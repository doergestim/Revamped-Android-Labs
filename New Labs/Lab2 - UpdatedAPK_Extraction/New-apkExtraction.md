# Extracting an APK for static analysis


#### This lab is the UPDATED VERSION
<hr>

If your device isn't booted yet, launch it with the following, look at [Lab Setup](/New%20Labs/NewLab_Setup.md/#Launching%the%Device)

First, let's launch the Android emulator. If your emulator is already running, you can skip this step.

Start by opening a terminal:

![](/New%20Labs/attachments/terminalinubuntu.png)

Then run the following to launch Android Studio:
<pre>android-studio</pre>

Once it launches, you will see the following window.<br>
Click `More Actions` and then `Virtual Device Manager`.

![](/New%20Labs/attachments/androidWelcomePage.png)

Then, you will see the following window.<br>
Click the `Play` icon next to the `Pixel 9` device to power it on.

![](/New%20Labs/attachments/turnondevice.png)

Behold! Your very own emulated Virtual Android!

![](/New%20Labs/attachments/devicewindow.png)

## Downloading the APK

We need to start by getting the package name for the application. Let's open a new terminal: <br>

![](/New%20Labs/attachments/terminalinubuntu.png)

First let's create the **Lab folder** and download **fdroid**: 

```bash 
mkdir -p ~/Android/Labs/Lab2_APK_Extraction
cd ~/Android/Labs/Lab2_APK_Extraction 
wget https://f-droid.org/F-Droid.apk -O F-Droid.apk
```

<img width="787" height="328" alt="image" src="https://github.com/user-attachments/assets/8e39a5c4-a95a-4807-a593-b0631afd3ea5" />

Now that we have downloaded **F-Droid** on Ubuntu, let's install it on the Android emulator: 

```bash
adb install -r F-Droid.apk
```

<img width="787" height="108" alt="image" src="https://github.com/user-attachments/assets/c520be4f-d6c1-42c4-9528-2917732d768e" />

To find the package name, use Android's package manager utility, `pm`, to list installed packages and pipe the output to `grep` to find F-Droid.

```bash
adb shell pm list packages | grep fdroid
```

<img width="1039" height="118" alt="image" src="https://github.com/user-attachments/assets/222d80e2-2443-4d7d-8983-dff0ad56fb46" />

To extract the APK from the device, we first need its full path. Run the following command:

```bash
adb shell pm path org.fdroid.fdroid
```

Copy the APK path ending in **base.apk**, excluding the **"package:"** prefix, as shown below:

<img width="1045" height="102" alt="image" src="https://github.com/user-attachments/assets/581bf60a-6e8b-41e8-b03e-040895a6efda" />

Next, run the final command. Make sure you give it a different output name. For this instance we'll use **"analyze-me.apk"** : 

```bash
adb pull [PATH_TO_APP] analyze-me.apk
```

<img width="1054" height="126" alt="image" src="https://github.com/user-attachments/assets/beae059a-9267-4783-8b48-73ef0ef334b3" />

## Running MobSF

First, check if Docker is installed:

```bash
sudo docker --version
```

If Docker is not installed, run the following commands:

```bash
sudo apt update
sudo apt install -y docker.io
sudo systemctl enable --now docker
```

<img width="1900" height="555" alt="image" src="https://github.com/user-attachments/assets/db1152bf-3840-478a-976c-4f4d64db8fd8" />

[MobSF](https://mobsf.github.io/docs/#/) can be run in a Docker container on your VM. Run the following command:

```bash
sudo docker run -it --rm -p 127.0.0.1:8000:8000 opensecurity/mobile-security-framework-mobsf:latest
```

<img width="1132" height="602" alt="image" src="https://github.com/user-attachments/assets/86ad2d7c-b33f-4036-ab9d-cc451423d07c" />

Now navigate to http://127.0.0.1:8000 in your VM web browser.

<img width="1080" height="681" alt="image" src="https://github.com/user-attachments/assets/f86e26ba-2eae-4094-b901-9c4f64b62d9e" />

Enter the VM password when prompted: 

<img width="693" height="590" alt="image" src="https://github.com/user-attachments/assets/15845bab-0700-4e0a-b5b4-e871d30e10aa" />

Once loaded, click "Upload & Analyze" and select the `analyze-me.apk` file from your lab directory.

<img width="839" height="651" alt="image" src="https://github.com/user-attachments/assets/f8879ac0-8149-4dad-a4cb-51c6e58fd378" />

<img width="1199" height="386" alt="image" src="https://github.com/user-attachments/assets/0d16aec3-d5af-471d-b747-abb58d8024a7" />

Processing the APK may take a while. Keep the MobSF container running and continue with the lab.

Once the analysis is complete, MobSF generates a static analysis report.

The report includes information about the application's permissions, exported components, signing certificates, and potential security issues.

<img width="1710" height="803" alt="image" src="https://github.com/user-attachments/assets/82d1b7a1-a3d9-4644-a231-09c4c4cbc5cf" />

We will explore these findings in more detail in the next lab, using a deliberately vulnerable Android application.

## Analyzing Other Apps
## Analyzing Other Apps

You can also analyze other Android applications using the emulator.

If your Android Virtual Device (AVD) includes the Google Play Store, you can download and install applications directly from it.

>[!NOTE]
>You will need to sign in with a Google account to download applications from the Play Store.

Alternatively, you can download an APK from a trusted source and install it using ADB.

First, navigate to your lab directory:

```bash
cd ~/Android/Labs/Lab2_APK_Extraction
```

Then install the downloaded APK:

```bash
adb install application.apk
```

Replace `application.apk` with the name of your downloaded APK file.

Once installed, follow the previous steps to locate and extract the application's APK for analysis.

>[!NOTE]
>Some applications use multiple APK files. If `adb shell pm path` returns multiple paths, extracting only `base.apk` may not capture the complete application.

***                                                                 

<b><i>Continuing the course? </br>[Next Lab](/New%20Labs/Lab3%20-%20MobSFlive/instructions.md)</i></b>

<b><i>Want to go back? </br>[Previous Lab](/New%20Labs/Lab1%20-%20UpdatedADB_Cheatsheet/New-adbCheatsheet.md)</i></b>

<b><i>Looking for a different lab? </br>[Lab Directory](/navigation.md)</i></b>
