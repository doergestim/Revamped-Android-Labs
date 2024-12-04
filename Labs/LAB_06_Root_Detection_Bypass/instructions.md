## Bypassing Root Detection using Frida

I CANNOT STRESS THIS ENOUGH!!!!!!

REMEMBER TO POWER OFF YOUR VM WHEN NOT WORKING ON LABS!!!!

Corellium WILL CHARGE YOU!!!!!

There are two potential methods of bypassing root detection, statically, by removing the relevant code and recompiling the apk, or dynamically.
In this lab, we will bypass the root detection at runtime using Frida.
The target application is `Hacker Bank Mobile`.

Let's install that app now.

Please download the following file.

```https://github.com/strandjs/IntroLabs/blob/master/IntroClassFiles/Tools/final.apk```

We also need to ensure the VPN is up and running.

<img width="637" alt="Screenshot 2023-12-30 at 1 57 21 PM" src="https://github.com/deruke/AndroidLabs/assets/22796374/abc319aa-b287-4113-974c-6d8b37c646a6">

Now let's install the application on our phone.

First, lets connect via adb.

`adb connect 10.11.1.1:5001 `


Next, let's install the app on our phone by pushing it through adb.

```adb install final.apk```

<img width="315" alt="Screenshot 2023-12-30 at 2 05 02 PM" src="https://github.com/deruke/AndroidLabs/assets/22796374/421c3f57-5dca-48d6-a596-294ed2c58f12">


Opening the app on your phone we see that we do not have the option to do anything other than aknowledge the alert, which consequently closes the application.
![screenshot](images/ss0.png)

**The following Commands are for reference only and do not need to be run on the Corellium device since it has the frida server pre-installed.**

<!--Typically, we would need to upload and start the Frida server on the device we are testing. That can be accomplished by restarting adb as root, and running the following commands:

* `adb push ~/Downloads/frida-server-15.2.2-android-x86 /data/local/tmp/frida-server`

* `adb shell "chmod 755 /data/local/tmp/frida-server"`

* `adb shell "/data/local/tmp/frida-server &"`
**End of reference commands.**-->

We will use [This script](https://codeshare.frida.re/@dzonerzy/fridantiroot/) from Frida Codeshare to bypass root detection on our target.

But first we need to install Frida on the phone.  On the left side of your device screen in Corellium select Frida.

<img width="89" alt="Screenshot 2023-12-30 at 2 19 08 PM" src="https://github.com/deruke/AndroidLabs/assets/22796374/6ad475b2-ebb2-42e8-b805-9f1a620e8656">

Next, select "Select a Process".

<img width="217" alt="Screenshot 2023-12-30 at 2 20 58 PM" src="https://github.com/deruke/AndroidLabs/assets/22796374/6f112142-bfb1-4416-9599-d31794075f67">

Wait till it loads the processes then select TheHackerBank.


<img width="291" alt="Screenshot 2023-12-30 at 2 22 07 PM" src="https://github.com/deruke/AndroidLabs/assets/22796374/3a407b63-be38-4900-9a90-86d900590e8b">

Then select ATTACH.



You can either download the script, and run it with the `-l` option, or, run it directly from the website with ` --codeshare dzonerzy/fridantiroot`

Run the following command from a terminal on your VM:
 `frida --codeshare dzonerzy/fridantiroot -U -f com.bhis.thehackerbank`

 the `-U` option tells frida to connect to the usb device (in our case the emulator "appears" as a usb device).

 The `-f` option specifies the target application ID that we want to load. The application should **not** be running already. the `-f` flag will spawn the process.

The command output should look exactly like the screenshot below. Make sure to enter `y` when prompted if you would like to trust the project.
![screenshot](images/ss1.png)

Looking back to your emulator screen you will now notice that the app has been launched, but this time root detection was not triggered.
![screenshot](images/ss2.png)

Your username is `notahacker` as shown, and your password is `654321`
