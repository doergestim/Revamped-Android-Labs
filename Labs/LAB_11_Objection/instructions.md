# Static & Dynamic Analysis with Objection #
Objection is a mobile exploration and exploitation framework built on top of Frida. It provides many helpful tools for inspecting device memory and processes, as well as displaying classes and activities, things which typically fall under the category of static analysis.

In this lab we are going walk through using objection to analyze our metasploit patched APK.

## Patching the APK ##
Before you can use any of the objection commands on an Android application, the application's APK itself needs to be patched and code signed to load the frida-gadget.so on start. 


First connect to the device using adb (this is necessary for objection to determine the architecture) and then run the following command:  
`objection patchapk -2 --source app.apk`

The `-2` tells objection to pass the `--use-aapt2` flag to apktool. This is necesary with many newer apps.

![](images/apkpatch.png)

Then use `adb install` to install the patched version of the APK to the device. (You will need to uninstall the pervious version first)

Next we launch the app and run `frida-ps -Ua` to list all running applications on the device. This shows us the process ID of our target app.

![](images/fridaps.png)

Enter the Objection REPL using the following command:
`objection -g <pid> explore`

![](images/repl.png)

From here we can explore the filesystem, upload and download files, and explore application components and classes.

Running `android shell_exec whoami` will execute the `whoami` command on thedevice. You are running as the application and will therefore see the applications user ID. (remember from previously that all apps are a linux user.)

By default, objection will start up in the main application build path. Running the `env` command. This will show the locations of the applications Files, Caches and other directories:internal storage of the application.

![](images/env.png)

You can enter some typical unix commands such as `ls` and `pwd` into the REPL as well.

`file download <remote path> <local path>`
`file upload <local path> <remote path>`

There are also some built in scripts to do things such as bpyass ssl pinning

![](images/05.png)
You might notice that on the hacker bank, this does not work. Objection is simply using fridascripts under the hood, and we need to find a different frida script.

If you wish to run a command on the host os, preface the command with a `!` for example: `!ls`


keystore analysis

Memory forensics.

Static Analysis Revamped

`android hooking list activities`

can also be used to list services and receivers.

## Spying On A Class or Method ##
To get a list of classes run the following command
`android hooking search classes com.bhis.thehackerb
ank`
`android hooking watch class_method asvid.github.io.fridaapp.MainActivity.sum --dump-args --dump-backtrace --dump-return`

## Logging ##

All commands issued,along with the output generated is logged to files on the host machine at the following locations:
* `~/.objection/objection.log`
* `~/.objection/objection_history`