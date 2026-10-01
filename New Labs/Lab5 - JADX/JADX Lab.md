# JADX

## In this lab we will

- Install JADX and download the training app on Ubuntu.
- Open and decompile an Android app in JADX.
- Read the app's basic information and permissions.
- Search its code and follow a method call.
- Examine three examples of insecure code.
- Export the readable code to a folder.

## What are JADX and DIVA?

**JADX** is a tool that turns compiled Android code into readable, Java-like code. This lets us examine how an app works. The reconstructed code may look different from the developer's original source.

We will examine **DIVA**, short for **Damn Insecure and Vulnerable App**, created by **Payatu**, a cybersecurity company. DIVA is a training app containing deliberate security mistakes, such as passwords written directly into code and sensitive information saved without encryption.

The file `diva-beta.apk` is DIVA's **APK**, the package used to distribute an Android app. It contains the compiled program and supporting files.

You will install the tools and download DIVA in the first two sections. Opening the APK in JADX lets us inspect it without running the Android app; an Android emulator is not required.

## 1. Install the tools

Run each command below in the Terminal. Press **Enter** after each command and wait for the prompt to return before continuing.

1. Refresh the list of available packages:

   ```bash
   sudo apt update
   ```

2. Install Java and the tools needed to download and extract the lab files:

   ```bash
   sudo apt install default-jre curl unzip ca-certificates tar gzip
   ```

   If asked **Continue?**, type **Y** and press **Enter**. Wait for installation to finish.

   `default-jre` installs the **Java runtime** needed by JADX. `curl` downloads files, `ca-certificates` supports HTTPS certificate verification, `unzip` extracts JADX's ZIP file, and `tar` with `gzip` extracts DIVA's archive.

3. Create a download folder and download **JADX 1.5.6** from its official GitHub release:

   ```bash
   mkdir -p "$HOME/Downloads"
   cd "$HOME/Downloads"
   curl --fail --location --output jadx-1.5.6.zip https://github.com/skylot/jadx/releases/download/v1.5.6/jadx-1.5.6.zip
   ```

   Wait for the download to finish. Using this specific version keeps the interface consistent with the walkthrough.

4. Extract JADX into `/opt/jadx-1.5.6`, a folder for the installed program:

   ```bash
   sudo unzip -o jadx-1.5.6.zip -d /opt/jadx-1.5.6
   sudo chmod +x /opt/jadx-1.5.6/bin/jadx /opt/jadx-1.5.6/bin/jadx-gui
   ```

5. Make both launchers available as Terminal commands:

   ```bash
   sudo ln -sf /opt/jadx-1.5.6/bin/jadx /usr/local/bin/jadx
   sudo ln -sf /opt/jadx-1.5.6/bin/jadx-gui /usr/local/bin/jadx-gui
   ```

   These commands create links so you can type `jadx` or `jadx-gui` from any folder. `jadx` is the command-line tool; `jadx-gui` opens the graphical interface.

6. Check that Java and JADX start successfully:

   ```bash
   java -version
   jadx --version
   ```

<img width="1785" height="881" alt="img1" src="https://github.com/user-attachments/assets/e297f5d1-8b45-461d-bfdc-a076e0ac668e" />

## 2. Download the DIVA APK

DIVA is the intentionally insecure Android app we will examine. We will download its packaged APK from Payatu's website and extract it into a lab folder.

1. In the same Terminal, create the folder and move into it:

   ```bash
   mkdir -p "$HOME/Desktop/Labs/JADX"
   cd "$HOME/Desktop/Labs/JADX"
   ```

2. Download the archive containing DIVA:

   ```bash
   curl --fail --location --output diva-beta.tar.gz https://www.payatu.com/wp-content/uploads/2016/01/diva-beta.tar.gz
   ```

3. Extract the APK:

   ```bash
   tar -xzf diva-beta.tar.gz
   ```

4. Confirm that the APK is present:

   ```bash
   ls -lh diva-beta.apk
   ```

   You should see `diva-beta.apk` listed with a size of approximately **1.5 MB**. This is the file you will open in JADX.

## 3. Open and decompile the APK

> [!NOTE]
> The command below loads DIVA's APK into JADX. As you open the app's classes, JADX automatically converts their compiled Android code into readable, Java-like code. This conversion is called **decompilation**. The following sections walk you through examining that code.

1. Open the APK in JADX to begin decompilation:

   ```bash
   jadx-gui diva-beta.apk
   ```

2. Wait for the JADX window to open and finish loading.

The left panel lists the APK's contents. **Source code** contains the reconstructed program, and **Resources** contains supporting files. Opening an item displays it in the larger panel on the right.

<img width="1593" height="987" alt="img2" src="https://github.com/user-attachments/assets/3d30fb18-a187-43d7-9c09-a89686d502f5" />

## 4. Read the Android manifest

The **manifest** is the app's configuration file. It identifies the app, lists requested permissions, and declares components such as its screens.

1. Click **Navigation** in the top menu.
2. Click **Go to AndroidManifest.xml**.
3. Scroll to the beginning of the file.
4. Locate the fields marked in the screenshot and read them alongside the table below.

<img width="1648" height="954" alt="img3" src="https://github.com/user-attachments/assets/aca082be-90ea-4837-bd5e-94887c364520" />

| Marker | What you should see | What it tells us |
| --- | --- | --- |
| 1 | `package="jakhar.aseem.diva"` | The app's package identifier. |
| 2 | `minSdkVersion="15"`, `targetSdkVersion="23"` | Its minimum Android API level and target API level. This is an older training app. |
| 3 | `WRITE_EXTERNAL_STORAGE`, `READ_EXTERNAL_STORAGE`, `INTERNET` | The permissions requested by this app. |
| 4 | `debuggable="true"`, `allowBackup="true"` | Debugging and backup settings worth inspecting during a security review. Their effect depends on the device and Android version. |
| 5 | `jakhar.aseem.diva.MainActivity`, with `MAIN` and `LAUNCHER` | The activity Android uses when someone launches the app normally. An activity usually represents an app screen. |

These settings give us an initial overview.

## 5. Find a class and read a hardcoded access key

A **class** groups related code. We will open `HardcodeActivity`, the class for DIVA's first hardcoded-key example.

1. Click **Navigation → Class search**.
2. Click the **Search for text** box and enter:

   ```text
   HardcodeActivity
   ```

3. Double-click the result named `jakhar.aseem.diva.HardcodeActivity`.

JADX decompiles the class and displays its reconstructed code in the main viewing area. This is the readable output of the decompilation step. You can see:

```java
public void access(View view)
```

This begins a **method**, a named block of code that performs an action. Inside it, locate:

```java
if (hckey.getText().toString().equals("vendorsecretkey")) {
```

Read the line from left to right:

1. `hckey` refers to the input field.
2. `getText().toString()` reads what the user entered as text.
3. `equals("vendorsecretkey")` compares that text with the key stored in the app.

When the text matches, the next line displays an **Access granted** message. Otherwise, the `else` section displays **Access denied**.
The key is **`vendorsecretkey`**. The comparison is case-sensitive and does not remove extra spaces. Because the key is included directly in the app's code, someone with the APK can recover it using JADX.

## 6. Search all code and follow a method call

We will now search for a log message without knowing which class contains it.

1. Click **Navigation → Text search**.
2. Replace the text in **Search for text** with:

   ```text
   diva-log
   ```
3. Wait for the result in `jakhar.aseem.diva.LogActivity`.
4. Double-click that result to open the matching code.

<img width="1538" height="1022" alt="img4" src="https://github.com/user-attachments/assets/0872d8e4-4fed-48cc-941e-6ee5b4749065" />

Look at the method called `checkout()`. It reads an input field and calls another method:

```java
processCC(cctxt.getText().toString());
```

1. Read its short body at the bottom of the file. It creates a `RuntimeException` and throws it. An exception interrupts normal execution and sends control to a matching error handler.
2. Look back at the `checkout()` method, immediately below it at the `catch` section, which handles this exception.

Inside that section, find:

```java
Log.e("diva-log", "Error while processing transaction with credit card: " + cctxt.getText().toString());
```

`Log.e` writes an error-level log entry. `diva-log` is its identifying tag. The `+` joins the message to the text entered in the credit-card field.

This means the code would place the full entered value in a log message. We can see the data flow by reading the program; we have not run the app or viewed an actual device log. Who can access those logs is a separate question that depends on the Android environment.

## 7. Read code that saves credentials

1. Click **Navigation → Class search**.
2. Replace the search text with:

   ```text
   InsecureDataStorage1Activity
   ```

3. Double-click `jakhar.aseem.diva.InsecureDataStorage1Activity`.
4. Scroll to the method named **`saveCredentials`**.

Near the beginning of the method, you will see **SharedPreferences**. This is Android storage for small pieces of data saved as named values.

Now locate these three lines:

```java
spedit.putString("user", usr.getText().toString());
spedit.putString("password", pwd.getText().toString());
spedit.commit();
```

Read them in order:

1. The first call saves the username under the name **`user`**.
2. The second saves the password under **`password`**.
3. `commit()` writes the changes to storage.

There is no encryption step between reading the fields and saving their values. The code therefore saves the password as plaintext.

> [!IMPORTANT]
> The APK shows **how** a password will be saved. It does not contain a password that a user might enter later.

The stored data normally belongs to the app's private area, so this finding does not mean every other app can read it automatically.

## 8. Export the readable code

Earlier, you used `jadx-gui` to decompile and inspect the app in the graphical interface. Here, you will use the command-line program `jadx` to decompile the APK and **save the resulting code and resources to a folder**.

1. Close the JADX window, don't save any changes andreturn to the Terminal. The export command works independently of the GUI.
2. Enter:

   ```bash
   cd "$HOME/Desktop/Labs/JADX"
   jadx -d exported diva-beta.apk
   ```

   The `-d exported` option tells JADX to write its output into a folder named `exported`.

3. Wait for the export to finish. You should see **`INFO - done`**.
4. List the new folder's contents:

   ```bash
   ls exported
   ```

You should see two directories:

```text
resources  sources
```

The **`sources`** directory contains the reconstructed Java files. The class we examined earlier is saved as:

```text
exported/sources/jakhar/aseem/diva/HardcodeActivity.java
```

The **`resources`** directory contains decoded supporting files, including `AndroidManifest.xml`. These exported files can be opened in a text editor outside JADX.

<img width="1397" height="867" alt="img5" src="https://github.com/user-attachments/assets/9ebca261-2eff-4ad2-8135-f4f3e2fb8927" />

---

<b><i>Continuing the course? </br>[Next Lab](../Lab6%20-%20RootDetectionBypass/instructions.md)</i></b>

<b><i>Want to go back? </br>[Previous Lab](../Lab4%20-%20UpdatedBurp_ProxySetup/New-BurpProxySetup.md)</i></b>

<b><i>Looking for a different lab? </br>[Lab Directory](/navigation.md)</i></b>

