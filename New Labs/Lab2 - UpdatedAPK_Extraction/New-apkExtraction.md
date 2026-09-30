# Extracting an APK for static analysis


#### This lab is the UPDATED VERSION
<hr>

If your device isn't booted yet, launch it with the following, look at [Lab Setup](/New%20Labs/NewLab_Setup.md/#Launching%the%Device)


## Downloading the APK

We need to start by getting the package name for the application.<br>

To find the package name, use Android's package manager utility, `pm`, to list all of the package names and pipe the output to `grep` in order to search for the package that you are testing.

```bash
adb shell pm list packages | grep fdroid
```

![](/New%20Labs/attachments/Lab1/listPackages.png)

To download the APK from the device you will need the applications full path. To get it,  run the following command:

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

[MobSF](https://mobsf.github.io/docs/#/) is already installed on your VM. To run the docker container, run this command:

```bash
sudo docker run -it --rm -p 8000:8000 opensecurity/mobile-security-framework-mobsf:latest
```

Enter the VM password if/when prompted.

![](/Labs/LAB_02_APK_Extraction/images/mobsflaunch.png)

Now navigate to http://0.0.0.0:8000 in your VM web browser.

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
