# Static Analysis with MobSF #

I CANNOT STRESS THIS ENOUGH!!!!!!

REMEMBER TO POWER OFF YOUR VM WHEN NOT WORKING ON LABS!!!!

Corellium WILL CHARGE YOU!!!!!

In this lab we will be using MobSF to analyze an APK. You can also use the app uploaded for analysis in the "APK_Extraction" lab, or look at one of the pre-generated reports in the course materials section. As an example we will be using the F-Droid app. You are encouraged to repeat this analysis using TheHackerBank app later in the class. Your goal is to analyze ther results and note down anything that stands out for later use.

Let's load up an app and look at it.

In your VM web browser open a tab to http://0.0.0.0:8000

Next, lets Upload and Analyze a file.

<img width="1426" alt="Screenshot 2023-12-30 at 11 53 30 AM" src="https://github.com/deruke/AndroidLabs/assets/22796374/2bba5969-4694-4ddc-b304-ce717c7a3f1a">

Please select base.apk in your home directory.

It may take a while for the file to finish Analyzing.

After a while you can select RECENT SCANS in the upper left corner.

You should see a status of the scan.

<img width="1415" alt="Screenshot 2023-12-30 at 11 57 11 AM" src="https://github.com/deruke/AndroidLabs/assets/22796374/a12a18d7-5270-4e34-acfb-166290adb2ee">

Please select Static Report from the right side.

<img width="156" alt="Screenshot 2023-12-30 at 11 58 16 AM" src="https://github.com/deruke/AndroidLabs/assets/22796374/53556720-9579-456f-b269-932ca1824e6b">


The first thing worth paying special attention to is the overview of the app components.
<!--![](images/exported.png)-->

<img width="1265" alt="Screenshot 2023-12-30 at 11 59 46 AM" src="https://github.com/deruke/AndroidLabs/assets/22796374/023ac466-ccd5-4b5a-9d7e-4d32d51595e3">

Specifically, the "exported" activities, services, receivers, and providers. The exported keyboard means they can be launched from outside the app and may present additional entry points or other attack vectors. We can further analyze the exported components by looking at the manifest file, and if necessary the decompiled code.

![](images/decomp.png) 
Here you can download the decompiled java, download the smali, (smali is the disassembled version of the code)

### Permissions ###
Permissions are gathered from the android manifest file. Each permission is marked with a status, most commonly "dangerous" or "normal". 
<!--![](images/permissions.png)-->

<img width="1415" alt="Screenshot 2023-12-30 at 12 00 58 PM" src="https://github.com/deruke/AndroidLabs/assets/22796374/e62582bc-1d17-48cf-b121-3998d41b8663">

Whether a permission is actually dangerous depends on the context of the application, and whether they make sense given the functionallity of the app. Permissions are not the most useful thing for a tester, but could be critical for malware analysis.
### Recon ###
The reconnaissance section is especially useful for gaining a better overall understanding of the application. First lets look at URLs. We can use this to see where the application is making connections to, potentially for malware analysis or testing our access outside of the app.

<!--![](images/urls.png)-->


<img width="1268" alt="Screenshot 2023-12-30 at 12 03 44 PM" src="https://github.com/deruke/AndroidLabs/assets/22796374/f434f21a-5689-4516-9fe8-6284693653f2">

Next we can also look at strings. Android best practices recommend that instead of hardcoding strings into the XML (UI) files, they be inserted as a key value pairs into the "Strings.xml" file and referenced by the layout files. This creates a lot of noise and looking through everything is not likely to be a good use of time.

MobSF extracts potentially sensitive values from this file and displays them in the "Hardcoded Secrets" tab. You should verify the information displayed in this section is actually sensitive before reporting it.

<!--![](images/secrets.png)-->
<img width="485" alt="Screenshot 2023-12-30 at 12 05 39 PM" src="https://github.com/deruke/AndroidLabs/assets/22796374/057e6fa8-ae56-4e12-9b2d-662ecb0a7f12">


### API ###
This Section displays the android APIs that the app uses. This section contains a lot of noise, however it can be useful, especially when combined with the search feature.

<!--![](images/API.png)-->

<img width="955" alt="Screenshot 2023-12-30 at 12 07 08 PM" src="https://github.com/deruke/AndroidLabs/assets/22796374/d02fd5da-9b36-4223-b0d9-2097d89ee3a5">


For example, the screenshot above shows us everywhere the command execution API is used. The files displayed are clickable and will show you the API usage location in code.

### Browsable Activities ###
MobSF Extracts a list of "Browsable Activities". These are activities that can be triggered by a web browser to display data referenced by a link. These are worth attention since they can sometimes be leveraged by an attacker to perform web based or intent based attacks.

<!--![](images/browsableactivities.png)-->

<img width="1429" alt="Screenshot 2023-12-30 at 12 07 48 PM" src="https://github.com/deruke/AndroidLabs/assets/22796374/3e9bc03d-5a9d-4011-8370-c3bbaa14e926">

### Certificate Analysis ###

<!--![](images/certs.png)-->

<img width="1424" alt="Screenshot 2023-12-30 at 12 09 00 PM" src="https://github.com/deruke/AndroidLabs/assets/22796374/a915b191-061f-4a17-87ba-40236a244dab">

The certificate analysis section will report any known vulnerabilities in the signing schemes of the application. Make sure to validate these before reporting them, report the android versions affected, and maybe even the overall market share of the affected android versions.

