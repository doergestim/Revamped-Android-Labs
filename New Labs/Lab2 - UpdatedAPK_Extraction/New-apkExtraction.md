# Extracting an APK for static analysis


#### This lab is the UPDATED VERSION
<hr>

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

Enter the VM password if/when prompted.

![](/Labs/LAB_02_APK_Extraction/images/mobsflaunch.png)

Now navigate to http://127.0.0.1:8000 in your VM web browser.

![](/Labs/LAB_02_APK_Extraction/images/webbrowsernavigate.png)

Once loaded, go ahead and click "Upload and Analyze", and upload an APK file. 

![](/Labs/LAB_02_APK_Extraction/images/uploadandanalyze.png)

![](/Labs/LAB_02_APK_Extraction/images/openapk.png)

Processing the APK may take a while. Keep the MobSF container running and continue with the lab.

## Analyzing Other Apps
You can also test out any app you'd like from the Play Store. To do this you will need to install `OpenGApps`. This will give you access to the Google Play Store. This can be done from the "Apps" tab in Corellium.

![](/Labs/LAB_02_APK_Extraction/images/installopengapps.png)

>[!Note]
>You will need to log-in with a Google account before you can download any apps

Then you can simply install apps from the Play Store as you normally would.

***                                                                 

<b><i>Continuing the course? </br>[Next Lab](/New%20Labs/Lab3%20-%20MobSFlive/instructions.md)</i></b>

<b><i>Want to go back? </br>[Previous Lab](/Labs/LAB_01_adb_basics/adb_cheatsheet.md)</i></b>

<b><i>Looking for a different lab? </br>[Lab Directory](/navigation.md)</i></b>
