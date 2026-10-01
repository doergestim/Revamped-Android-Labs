# HTTP Toolkit Setup Commands

## Installing Tools

```bash
sudo apt install docker.io -y
sudo apt install docker-compose -y
sudo apt install net-tools -y
```

## HttpToolKit Installation

```bash
cd ~/Android/Sdk
wget https://github.com/httptoolkit/httptoolkit-desktop/releases/download/v1.27.2/HttpToolkit-1.27.2-x64.deb
mv HttpToolkit-1.27.2-x64.deb /tmp/
sudo apt install /tmp/HttpToolkit-1.27.2-x64.deb
```

## Setup Folders

```bash
cd ~/Android/Sdk
mkdir HttpToolKit_Lab
cd HttpToolKit_Lab
```

## Backend Files

```bash
cd ~/Android/Sdk/HttpToolKit_Lab
mkdir backend
cd ./backend

cat > requirements.txt <<'EOF'
fastapi==0.115.6
uvicorn==0.34.0
bcrypt==4.2.1
PyJWT==2.10.1
EOF

cat > compose.yaml <<'EOF'
services:
  api:
    build: .
    ports:
      - "${LAB_API_PORT:-8081}:8000"
    volumes:
      - ./data:/app/data
    environment:
      LAB_DB: /app/data/lab.db
      LAB_JWT_SECRET: ${LAB_JWT_SECRET:-local-training-only-secret-change-if-shared}
    restart: unless-stopped
EOF

cat > Dockerfile <<'EOF'
FROM python:3.12-slim
WORKDIR /app
COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt
COPY app ./app
EXPOSE 8000
CMD ["uvicorn", "app.main:app", "--host", "0.0.0.0", "--port", "8000"]
EOF

mkdir app
cd ./app

cat > main.py <<'EOF'
import json
import re
import sqlite3
import time
from contextlib import asynccontextmanager

import bcrypt
from fastapi import Depends, FastAPI, HTTPException, Request
from pydantic import BaseModel, Field

from .auth import current_user_id, make_token
from .database import connection, initialize


@asynccontextmanager
async def lifespan(app: FastAPI):
    initialize()
    yield


app = FastAPI(title="HTTP Toolkit Lab API", lifespan=lifespan)
EMAIL_PATTERN = re.compile(
    r"^[A-Za-z0-9._%+-]+@[A-Za-z0-9-]+(?:\.[A-Za-z0-9-]+)*\.[A-Za-z]+$"
)


def normalized_email(value: str) -> str:
    email = value.strip().lower()
    if len(email) > 120 or not EMAIL_PATTERN.fullmatch(email):
        raise HTTPException(422, "Enter an email like name@example.com")
    return email


@app.middleware("http")
async def request_log(request: Request, call_next):
    start = time.monotonic()
    response = await call_next(request)
    # Uvicorn logs method, path and status; no request body or Authorization header.
    response.headers["X-Lab-Duration-Ms"] = str(round((time.monotonic() - start) * 1000))
    return response


class Register(BaseModel):
    username: str = Field(min_length=2, max_length=40)
    email: str = Field(min_length=3, max_length=120)
    password: str = Field(min_length=8, max_length=128)


class Login(BaseModel):
    email: str
    password: str


class NoteInput(BaseModel):
    title: str = Field(min_length=1, max_length=100)
    content: str = Field(min_length=1, max_length=1000)


def public_user(row):
    return {key: row[key] for key in ("id", "username", "email", "bio", "role")}


def require_user(db, user_id):
    row = db.execute("SELECT id,username,email,bio,role FROM users WHERE id=?", (user_id,)).fetchone()
    if row is None:
        raise HTTPException(401, "Account no longer exists")
    return row


@app.post("/api/register", status_code=201)
def register(body: Register):
    username, email = body.username.strip(), normalized_email(body.email)
    if len(username) < 2:
        raise HTTPException(422, "Invalid username")
    password_hash = bcrypt.hashpw(body.password.encode(), bcrypt.gensalt()).decode()
    try:
        with connection() as db:
            cursor = db.execute("INSERT INTO users(username,email,bio,role) VALUES (?,?,?,?)",
                                (username, email, "", "user"))
            db.execute("INSERT INTO credentials(user_id,password_hash) VALUES (?,?)",
                       (cursor.lastrowid, password_hash))
            user_id = cursor.lastrowid
    except sqlite3.IntegrityError:
        raise HTTPException(409, "Username or email already exists") from None
    return {"message": "Registration successful", "user_id": user_id}


@app.post("/api/login")
def login(body: Login):
    with connection() as db:
        row = db.execute("""SELECT u.id,u.username,u.role,c.password_hash FROM users u
                            JOIN credentials c ON c.user_id=u.id WHERE u.email=?""",
                         (body.email.strip(),)).fetchone()
    # Same response for missing account and wrong password.
    if row is None or not bcrypt.checkpw(body.password.encode(), row["password_hash"].encode()):
        raise HTTPException(401, "Invalid credentials")
    return {"token": make_token(row["id"]), "user_id": row["id"],
            "username": row["username"], "role": row["role"]}


@app.get("/api/profile")
def profile(user_id: int = Depends(current_user_id)):
    with connection() as db:
        return public_user(require_user(db, user_id))


@app.put("/api/profile")
async def update_profile(request: Request, user_id: int = Depends(current_user_id)):
    # INTENTIONAL LAB FLAW: role is accepted even though the UI edits only
    # username, email and bio. Never extend this to credentials or the id.
    try:
        body = await request.json()
    except (json.JSONDecodeError, UnicodeDecodeError):
        raise HTTPException(422, "Invalid JSON") from None
    if not isinstance(body, dict) or not body:
        raise HTTPException(422, "Expected a non-empty JSON object")
    allowed = {"username", "email", "bio", "role"}
    unknown = set(body) - allowed
    if unknown:
        raise HTTPException(422, f"Unknown field: {sorted(unknown)[0]}")
    if any(not isinstance(v, str) for v in body.values()):
        raise HTTPException(422, "Fields must be strings")
    if "role" in body and body["role"] not in ("user", "admin"):
        raise HTTPException(422, "Invalid role")
    if "username" in body and not (2 <= len(body["username"].strip()) <= 40):
        raise HTTPException(422, "Invalid username")
    if "bio" in body and len(body["bio"]) > 500:
        raise HTTPException(422, "Bio is too long")
    for key in ("username", "email"):
        if key in body:
            body[key] = body[key].strip()
    if "email" in body:
        body["email"] = normalized_email(body["email"])
    # Column names come only from the fixed allowlist, never from arbitrary input.
    fields = ", ".join(f"{key}=?" for key in body)
    try:
        with connection() as db:
            require_user(db, user_id)
            db.execute(f"UPDATE users SET {fields} WHERE id=?", (*body.values(), user_id))
    except sqlite3.IntegrityError:
        raise HTTPException(409, "Username or email already exists") from None
    return {"message": "Profile updated successfully"}


@app.get("/api/users/{requested_id}")
def get_user(requested_id: int, user_id: int = Depends(current_user_id)):
    # INTENTIONAL LAB FLAW (IDOR/BOLA): validates token but does not compare
    # requested_id to user_id. No credential fields are selected.
    with connection() as db:
        require_user(db, user_id)
        row = db.execute("SELECT id,username,email,bio,role FROM users WHERE id=?", (requested_id,)).fetchone()
    if row is None:
        raise HTTPException(404, "User not found")
    return public_user(row)


@app.get("/api/notes")
def notes(user_id: int = Depends(current_user_id)):
    with connection() as db:
        require_user(db, user_id)
        rows = db.execute("SELECT id,title,content FROM notes WHERE user_id=? ORDER BY id", (user_id,)).fetchall()
    return [dict(row) for row in rows]


@app.post("/api/notes", status_code=201)
def post_note(body: NoteInput, user_id: int = Depends(current_user_id)):
    with connection() as db:
        require_user(db, user_id)
        cursor = db.execute("INSERT INTO notes(user_id,title,content) VALUES (?,?,?)",
                            (user_id, body.title, body.content))
        return {"id": cursor.lastrowid, "title": body.title, "content": body.content}


@app.get("/api/status")
def status():
    return {"status": "online", "service": "HTTP Toolkit Lab API", "version": "1.0"}
EOF

cat > database.py <<'EOF'
import os
import sqlite3
from contextlib import contextmanager
from pathlib import Path

import bcrypt

DB_PATH = Path(os.getenv("LAB_DB", str(Path(__file__).resolve().parents[1] / "data" / "lab.db")))
SEED_USERS = [
    (1, "student", "student@lab.local", "Cybersecurity student", "user"),
    (2, "alex", "alex@lab.local", "Android developer", "user"),
    (3, "maria", "maria@lab.local", "Security analyst", "user"),
    (4, "admin", "admin@lab.local", "Administrator", "admin"),
]


@contextmanager
def connection():
    db = sqlite3.connect(DB_PATH, timeout=10)
    db.row_factory = sqlite3.Row
    db.execute("PRAGMA foreign_keys = ON")
    try:
        yield db
        db.commit()
    except Exception:
        db.rollback()
        raise
    finally:
        db.close()


def initialize():
    DB_PATH.parent.mkdir(parents=True, exist_ok=True)
    with connection() as db:
        db.executescript("""
            CREATE TABLE IF NOT EXISTS users (
                id INTEGER PRIMARY KEY,
                username TEXT NOT NULL UNIQUE COLLATE NOCASE,
                email TEXT NOT NULL UNIQUE COLLATE NOCASE,
                bio TEXT NOT NULL DEFAULT '',
                role TEXT NOT NULL DEFAULT 'user' CHECK (role IN ('user','admin'))
            );
            CREATE TABLE IF NOT EXISTS credentials (
                user_id INTEGER PRIMARY KEY REFERENCES users(id) ON DELETE CASCADE,
                password_hash TEXT NOT NULL
            );
        """)
        # Preserve notes created by earlier versions of the lab.
        if db.execute("SELECT 1 FROM sqlite_master WHERE type='table' AND name='messages'").fetchone():
            db.execute("ALTER TABLE messages RENAME TO notes")
            db.execute("ALTER TABLE notes RENAME COLUMN message TO content")
        db.execute("""CREATE TABLE IF NOT EXISTS notes (
            id INTEGER PRIMARY KEY,
            user_id INTEGER NOT NULL REFERENCES users(id) ON DELETE CASCADE,
            title TEXT NOT NULL,
            content TEXT NOT NULL
        )""")
        if db.execute("SELECT COUNT(*) FROM users").fetchone()[0] == 0:
            password_hash = bcrypt.hashpw(b"Password123!", bcrypt.gensalt()).decode()
            db.executemany("INSERT INTO users(id,username,email,bio,role) VALUES (?,?,?,?,?)", SEED_USERS)
            db.executemany("INSERT INTO credentials(user_id,password_hash) VALUES (?,?)",
                           [(row[0], password_hash) for row in SEED_USERS])
            db.executemany("INSERT INTO notes(user_id,title,content) VALUES (?,?,?)",
                           [(row[0], "Welcome", "Welcome to the HTTP Toolkit lab.") for row in SEED_USERS])
EOF

cat > auth.py <<'EOF'
import os
from datetime import datetime, timedelta, timezone

import jwt
from fastapi import Header, HTTPException

SECRET = os.getenv("LAB_JWT_SECRET", "local-training-only-secret-change-if-shared")
ALGORITHM = "HS256"


def make_token(user_id: int) -> str:
    return jwt.encode({"sub": str(user_id), "exp": datetime.now(timezone.utc) + timedelta(hours=12)},
                      SECRET, algorithm=ALGORITHM)


def current_user_id(authorization: str | None = Header(default=None)) -> int:
    if not authorization or not authorization.startswith("Bearer "):
        raise HTTPException(401, "Bearer token required")
    try:
        payload = jwt.decode(authorization[7:], SECRET, algorithms=[ALGORITHM])
        return int(payload["sub"])
    except (jwt.PyJWTError, ValueError, KeyError):
        raise HTTPException(401, "Invalid or expired token") from None
EOF
```

## APK Application

Install this application in the following folder:

```bash
cd ~/Android/Sdk/HttpToolKit_Lab
```

I have uploaded a file with the base64 format of the APK application.

Install the file in the above directory and then execute the following command:

```bash
base64 -d AndroidAPK.txt > AndroidLab.zip
unzip AndroidLab.zip
```
