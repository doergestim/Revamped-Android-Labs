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

*F-Droid* should now be visible in the emulator app menu. Let's check! 

In the android emulator, swipe up using your mouse to open the app menu. F-Droid should already be installed : 

<img width="533" height="867" alt="image" src="https://github.com/user-attachments/assets/db59c2b3-cad7-4026-a2b3-520c4bc0d070" />

<br>

<img width="461" height="819" alt="image" src="https://github.com/user-attachments/assets/cc742338-51a2-4113-8630-02778ebc7f36" />

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

If prompted to log in, use the default MobSF credentials:

- **Username:** `mobsf`
- **Password:** `mobsf` 

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

You can also extract APKs from applications installed directly through the Android interface.

Open **F-Droid** on your Android emulator and search for **Open Camera**.

<img width="464" height="829" alt="image" src="https://github.com/user-attachments/assets/ccde5723-68df-4593-9c46-e41d810edfe8" />

<br>

<img width="466" height="826" alt="image" src="https://github.com/user-attachments/assets/b40db286-b434-430d-9bdc-fa5d60002499" />

<br>

<img width="482" height="829" alt="image" src="https://github.com/user-attachments/assets/2c2e3ab4-c112-4d7c-8777-70c55bdbb873" />

<br>

<img width="469" height="825" alt="image" src="https://github.com/user-attachments/assets/fb1209d5-3cf2-47fe-8fe6-5b4dec0d624c" />

<br>

<img width="476" height="826" alt="image" src="https://github.com/user-attachments/assets/d880d041-2abc-4d4a-8bca-d497f4263aae" />

<br>

When prompted, open Settings and enable Allow from this source to let F-Droid install the application.
<img width="502" height="839" alt="image" src="https://github.com/user-attachments/assets/48d22de7-e457-4a08-b05b-2d44b5cc7daf" />

<br>

<img width="355" height="554" alt="image" src="https://github.com/user-attachments/assets/8a0a1a44-e49b-4561-9724-df1f833c8a8b" />

<br>

<img width="476" height="818" alt="image" src="https://github.com/user-attachments/assets/1e69e056-201b-4a66-92cc-df8fb20aea46" />

<br>

Once installed, return to your Ubuntu terminal and find the application's package name:

```bash
adb shell pm list packages | grep -i opencamera
```

<img width="1101" height="94" alt="image" src="https://github.com/user-attachments/assets/15f78ad2-d7a3-4b6b-9a46-fe8dc5d3c900" />

Find the installed APK's location:

```bash
adb shell pm path net.sourceforge.opencamera
```

<img width="1044" height="59" alt="image" src="https://github.com/user-attachments/assets/cb520839-faa9-480c-bca4-3773e569336c" />

Extract the APK using the path returned above, excluding the `package:` prefix:

```bash
adb pull [PATH_TO_APP] OpenCamera-extracted.apk
```

<img width="1861" height="59" alt="image" src="https://github.com/user-attachments/assets/ee96b73c-7fb3-4532-9f82-4c64bbef8714" />


You can now upload the extracted APK to MobSF for static analysis.

## Using ADB with Physical Android Devices

The same APK extraction process works with physical Android devices, including many applications installed through the Google Play Store.

To connect a physical Android device:

1. Enable **Developer Options** and **USB Debugging** on your phone.
2. Connect the phone to your computer using a USB data cable.
3. Accept the USB debugging authorization prompt on your phone.

Verify the connection:

```bash
adb devices -l
```

<img width="1211" height="117" alt="image" src="https://github.com/user-attachments/assets/da56da3a-20f2-4ec7-addf-8410605f60b0" />

For example, a physical Google Pixel 9 might appear as:

```text
List of devices attached
3A041FDF600ABC    device product:tokay model:Pixel_9 device:tokay transport_id:1
```

The device identifier will differ from the example above.

Once connected, you can use the same `pm list packages`, `pm path`, and `adb pull` commands to extract installed APKs.

>[!NOTE]
>Some applications use split APKs, meaning that extracting only `base.apk` might not provide the complete application. Extracting an APK does not include the application's private user data.


***                                                                 

<b><i>Continuing the course? </br>[Next Lab](/New%20Labs/Lab3%20-%20MobSFlive/instructions.md)</i></b>

<b><i>Want to go back? </br>[Previous Lab](/New%20Labs/Lab1%20-%20UpdatedADB_Cheatsheet/New-adbCheatsheet.md)</i></b>

<b><i>Looking for a different lab? </br>[Lab Directory](/navigation.md)</i></b>
