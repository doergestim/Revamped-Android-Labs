# Becoming an `adb` Ninja


#### This lab is the UPDATED VERSION
<hr>

If your device isn't booted yet, launch it with the following, look at [Lab Setup](/New%20Labs/NewLab_Setup.md/#Launching%the%Device)

To verify access to your Android device, use adb's devices command.<br>
Let's open a new terminal and run the following:

```bash
adb devices
```

![](/New%20Labs/attachments/Lab1/adbDevices.png)

With adb connection, you can gain an interactive shell on the device by running the following commands: 

```bash
adb shell
su
id
```

![](/New%20Labs/attachments/Lab1/adbShell.png)

This is useful if you're just starting to explore the app and you're not quite sure what you're looking for yet.

Go ahead and enter `exit` twice to leave the shell.

![](/New%20Labs/attachments/Lab1/exitShell.png)

If you already know exactly what you're looking for, you can also use adb's shell command to run commands interactively, even to pipe output to your local testing system. For example, maybe you're researching the security of Android's KeyChain. The following command will run on the Android device and pipe the output to your local VM. 

```bash
adb shell pm list packages | grep key
```

![](/New%20Labs/attachments/Lab1/grepKey.png)

When penetration testing mobile apps, it is possible that you will receive the APK files outside of the Google Play store, as a stand-alone APK file. In which case, you will likely use adb to install the app. This is easily accomplished with adb's install command. After installing the app, you can use adb shell to find the package name after the APK is installed.

***
# Installing third-party apps


Let's play with a third-party app called F-Droid.  Often times you will be given a .apk file to test.  This will walk through how to install those apps outside of the Google Play Store.

First, let's download it.

```bash
wget https://f-droid.org/F-Droid.apk
```

![](/New%20Labs/attachments/Lab1/wGet.png)

Next, let's install it.

```bash
adb install F-Droid.apk
```

![](/New%20Labs/attachments/Lab1/installFdroid.png)

If the app you are testing is from the Google Play store, then you will want to extract the APK from the device after installing the app. This will allow you to conduct static analysis of the app. To do so, you need to find out the name of the package and the full file path where the APK file is saved to.

To find the package name, use Android's package manager utility, `pm`, to list all of the package names and pipe the output to `grep` in order to search for the package that you are testing.

```bash
adb shell pm list packages | grep fdroid
```

![](/New%20Labs/attachments/Lab1/listPackages.png)

Use `pm` again, with the package name, to find the full path to the APK.

```bash
adb shell pm path org.fdroid.fdroid
```

![](/New%20Labs/attachments/Lab1/findPath.png)

>[!Note] 
>The path to the package will be different than what you see here. Each time an APK is installed, the directory path is randomly generated. As a demonstration of this, see the following screenshot where the app has been uninstalled and re-installed. 

Notice how the file paths change.

![](/New%20Labs/attachments/Lab1/filePathChanges.png)

The official explanation for why the name dynamically changes is because they hate you...

That long, messy string is the full file path that we will use to copy the APK file from the device. With adb's `pull` command, let's run the following:

```bash
adb pull /data/app/[Your unique directory string]/base.apk
```

>[!TIP] 
>To make this easier, you can highlight, right click and copy the directory path from the command that we ran earlier!

![](/New%20Labs/attachments/Lab1/copyString.png)

Now paste the path after `adb pull` 

![](/New%20Labs/attachments/Lab1/adbPullPath.png)

There will likely be occasions where you need to copy a file from your testing system to the Android device. For this situation, you can use adb's `push` command. When copying to a device, be mindful of where you are copying to. Due to Android's file system permissions, you might accidentally try to copy to a read-only location. 

A common location to copy files to is the `/sdcard/Download/` directory which does not require root access. 

![](/New%20Labs/attachments/Lab1/pushSdcard.png)

A common testing technique for mobile testing is to determine if sensitive information is being sent to the system logger. The developer may have accidentally left a misplaced debug statement, or perhaps they didn't consider the  threat of leaking sensitive information in the log during development. We can use adb's `logcat` command to stream the system logger.

When adb's `logcat` is ran without any arguments, it will print literally everything, which can make conducting meaningful analysis nearly impossible. When testing a mobile app, you will likely want to filter logcat's output. There are a few ways you can do this.

One way is to use some of logcat's built in filter mechanisms, which will filter depending on the log type (e.g. error, informational, debug, all ,etc.). Another way to filter logcat's output is by the package name as this will likely correspond to the process name. This is the method we are going to use. 

Let's run the following command:

```bash
adb logcat | grep fdroid
```

![](/New%20Labs/attachments/Lab1/grepFdroid.png)

You can use `ctrl + c` in order to get back to the prompt.

If you think that the app might be spawning new processes or making inter-process communication (IPC) calls, it might be worthwhile to filter by the process ID (PID) associated with the app that you're testing. To do this, you can use the `ps` command to find the PID associated with the app you're testing and use that as a filter for logcat.

Next, we need to launch Fdroid. Do so by running the following:

```bash
adb shell monkey -p org.fdroid.fdroid 1
```

>[!Note] 
>Remember, your PID will be different than what is shown below.

```bash
adb shell ps | head -n 1
adb shell ps | grep fdroid
adb logcat | grep [Pid]
```

![](/New%20Labs/attachments/Lab1/grepPid.png)

***                                                                
