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

<img width="793" height="350" alt="image" src="https://github.com/user-attachments/assets/698a9349-7f6f-4972-852e-909a6db30874" />

Now that we have downloaded **F-Droid** on Ubuntu, let's install it on the Android emulator: 

```bash
adb install -r F-Droid.apk
```

To find the package name, use Android's package manager utility, `pm`, to list installed packages and pipe the output to `grep` to find F-Droid.

```bash
adb shell pm list packages | grep fdroid
```

![](/New%20Labs/attachments/Lab1/listPackages.png)

To extract the APK from the device, we first need its full path. Run the following command:

```bash
adb shell pm path org.fdroid.fdroid
```

Copy the line in the output that ends in `base.apk` as shown below:

![](/New%20Labs/attachments/Lab2/copyBaseApk.png)

Next, run the final command. Make sure you give it a different output name this time!

```bash
adb pull [PATH_TO_APP] [OUTFILE_NAME]
```

![](/New%20Labs/attachments/Lab2/renameApk.png)

## Running MobSF

[MobSF](https://mobsf.github.io/docs/#/) can be run in a Docker container on your VM. Run the following command:

```bash
sudo docker run -it --rm -p 8000:8000 opensecurity/mobile-security-framework-mobsf:latest
```

Enter the VM password if/when prompted.

![](/Labs/LAB_02_APK_Extraction/images/mobsflaunch.png)

Now navigate to http://127.0.0.1:8000 in your VM web browser.

![](/Labs/LAB_02_APK_Extraction/images/webbrowsernavigate.png)

Once loaded, go ahead and click "Upload and Analyze", and upload an APK file. 

![](/Labs/LAB_02_APK_Extraction/images/uploadandanalyze.png)

![](/Labs/LAB_02_APK_Extraction/images/openapk.png)

Processing the APK file will take a while, so let's continue on. Let it keep running in the background as we will use it for future labs.

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
