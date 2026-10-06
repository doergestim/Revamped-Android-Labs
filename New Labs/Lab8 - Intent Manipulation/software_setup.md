# Software Setup

This section prepares the Ubuntu system for using Apktool.

Apktool is a Java application used to decode Android APK files into a readable project structure and to build them again after changes. On Linux, the recommended setup uses two files:

- `apktool`, a small wrapper script that lets you run Apktool directly from the terminal
- `apktool.jar`, the Java application itself

The commands below install Java, download both Apktool files, place them in `/usr/local/bin`, and verify that the installation works.

## 1. Update the package list + Install Java and wget

First, refresh the local package index so Ubuntu knows about the latest packages available. Apktool requires Java in order to run.


```bash
sudo apt update && sudo apt install -y default-jre wget
```

The `-y` option automatically confirms the installation prompt. Wait for the output, there should be no errors. 

<img width="915" height="534" alt="image" src="https://github.com/user-attachments/assets/7b3d7e7b-f3a7-48f5-9090-7407ed7ffcd9" />

## 2. Verify the Java installation

Check that Java is installed and available from the command line.

```bash
java -version
```

A successful installation should print the installed OpenJDK version and runtime information.

<img width="828" height="133" alt="image" src="https://github.com/user-attachments/assets/4379460e-0128-4bff-9601-de70bc4f16f8" />


## 3. Create a temporary setup directory

Create a separate directory for the Apktool installation files and move into it.

```bash
mkdir -p ~/apktool-setup
cd ~/apktool-setup
```

Verify the current directory if needed:

```bash
pwd
```

The output should end with **/apktool-setup**:

<img width="505" height="104" alt="image" src="https://github.com/user-attachments/assets/97bc0cae-69b0-41fc-8fb8-ad8845d0af87" />


## 4. Download the Linux Apktool wrapper script

Download the official Linux wrapper script.

```bash
wget https://raw.githubusercontent.com/iBotPeaches/Apktool/master/scripts/linux/apktool
```

The expected output is : 

<img width="1766" height="316" alt="image" src="https://github.com/user-attachments/assets/48fe7d73-8845-4d5e-a647-36a9b3666e31" />

The wrapper script is a small Bash script that launches `apktool.jar`. It allows Apktool to be run with a simple command such as (you do not need to run these commands yet):

```bash
apktool
```

instead of manually running:

```bash
java -jar apktool.jar
```

## 5. Download Apktool 3.0.3

Download the Apktool 3.0.3 JAR file.

```bash
wget https://github.com/iBotPeaches/Apktool/releases/download/v3.0.3/apktool_3.0.3.jar
```

Check that both files were downloaded successfully.

```bash
ls -lh apktool*
```

You should now see two files:

```text
apktool
apktool_3.0.3.jar
```

<img width="1831" height="627" alt="image" src="https://github.com/user-attachments/assets/3750ffc7-f5b9-43d3-9d08-ad68149f0689" />

## 6. Verify the Apktool JAR checksum

Verify the SHA-256 checksum of the downloaded JAR file.

```bash
sha256sum apktool_3.0.3.jar
```

For Apktool 3.0.3, the expected SHA-256 value is:

```text
dbf930b076c6b9be08d57c449cacefc3bdd6b71ebd59b3066fc0e1f5b14f9423
```

The calculated value should match the expected value before continuing.

<img width="819" height="71" alt="image" src="https://github.com/user-attachments/assets/3126dc22-5b5f-4deb-bccf-aa83abcaaf6d" />

## 7. Rename the Apktool JAR

Rename the downloaded JAR file to `apktool.jar`.

```bash
mv apktool_3.0.3.jar apktool.jar
```

Verify the new filenames.

```bash
ls -lh apktool*
```

The directory should now contain:

```text
apktool
apktool.jar
```

<img width="810" height="118" alt="image" src="https://github.com/user-attachments/assets/3c81dd61-df73-414f-beef-b29e1323fd50" />


## 8. Move Apktool to /usr/local/bin

Move both files into `/usr/local/bin`.

```bash
sudo mv apktool apktool.jar /usr/local/bin/
```

`/usr/local/bin` is normally included in the system `PATH`, so programs stored there can be executed without specifying their full filesystem path.

## 9. Make the files executable

Add executable permissions to both Apktool files.

```bash
sudo chmod +x /usr/local/bin/apktool
sudo chmod +x /usr/local/bin/apktool.jar
```
 
## 10. Verify the installed files

Check that the files are present in `/usr/local/bin` and have executable permissions.

```bash
ls -l /usr/local/bin/apktool*
```

The permissions should include `x`, indicating that the files are executable.

<img width="832" height="158" alt="image" src="https://github.com/user-attachments/assets/063fe0ac-8b2e-48b4-9f71-a9459305e665" />


## 11. Verify that Apktool is available in PATH

Use `which` to confirm which Apktool executable the shell will use.

```bash
which apktool
```

The expected output is:

```text
/usr/local/bin/apktool
```

<img width="731" height="82" alt="image" src="https://github.com/user-attachments/assets/e1a2f035-191c-4e62-8893-5685c0d65c9b" />


## 12. Verify the Apktool version

Run Apktool and print its installed version.

```bash
apktool --version
```

The expected version is:

```text
3.0.3
```

<img width="570" height="85" alt="image" src="https://github.com/user-attachments/assets/43812b19-24c0-406f-94ec-94db9358a091" />

## Final verification

The following commands can be used as a quick final check:

```bash
java -version
which apktool
apktool --version
```

A working setup should show:

- a valid Java runtime
- `/usr/local/bin/apktool`
- Apktool version `3.0.3`


<img width="821" height="209" alt="image" src="https://github.com/user-attachments/assets/4024fd3a-b19d-4a9c-8e7a-2523e52c82f9" />


## Cleanup 

Run these commands to remove the created directory : 

```bash
cd ~
rmdir ~/apktool-setup
```

The directory should not exist anymore :

<img width="683" height="153" alt="image" src="https://github.com/user-attachments/assets/126c1dfc-f225-4288-9bd6-8f4f455f733f" />


Now let's try to run **apktool**:

<img width="866" height="806" alt="image" src="https://github.com/user-attachments/assets/f71e0f71-b5b6-4312-9122-279bc0cf9852" />

Running apktool without additional arguments confirms that the installation is working correctly. The command starts Apktool successfully and displays the available commands and usage information.



# At this point, the system is ready to decode and rebuild APK files with Apktool! 




