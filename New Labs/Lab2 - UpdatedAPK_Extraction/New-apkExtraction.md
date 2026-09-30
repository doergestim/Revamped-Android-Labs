# Extracting an APK for static analysis


#### This lab is the UPDATED VERSION
<hr>

If your device isn't booted yet, launch it with the following, look at [Lab Setup](/New%20Labs/NewLab_Setup.md/#Launching%the%Device)


## Downloading the APK

Now that we are connected, the first thing you will need is the application ID of the app we are testing. We will be analyzing F-Droid. On the Corellium interface click on the “Apps” tab and start typing the name of the application. The application ID is found directly under the application name as shown below. 

![](/Labs/LAB_02_APK_Extraction/images/gettingapplicationid.png)

To download the APK from the device you will need the applications full path. To get it,  run the following command:

<pre>adb shell pm path org.fdroid.fdroid</pre>

Copy the line in the output that ends in `base.apk` as shown below:

![](/Labs/LAB_02_APK_Extraction/images/copybaseapk.png)

Next, run the final command. Make sure you give it a different output name this time!

<pre>adb pull [PATH_TO_APP] [OUTFILE_NAME]</pre>

![](/Labs/LAB_02_APK_Extraction/images/renameapk.png)

## Running MobSF

[MobSF](https://mobsf.github.io/docs/#/) is already installed on your VM. To run the docker container, run this command:

<pre>sudo docker run -it --rm -p 8000:8000 opensecurity/mobile-security-framework-mobsf:latest</pre>

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

<b><i>Continuing the course? </br>[Next Lab](/Labs/LAB_03_Static_Analysis/instructions.md)</i></b>

<b><i>Want to go back? </br>[Previous Lab](/Labs/LAB_01_adb_basics/adb_cheatsheet.md)</i></b>

<b><i>Looking for a different lab? </br>[Lab Directory](/navigation.md)</i></b>
