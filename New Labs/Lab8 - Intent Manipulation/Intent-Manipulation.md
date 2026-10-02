# Intent Manipulation

If your device isn't booted yet, launch it with the following, look at [Lab Setup](/New%20Labs/NewLab_Setup.md/#Launching%the%Device)

If your software is not downloaded yet, do it with the help of [Software Setup](./software_setup.md)

> [!IMPORTANT] 
> 
> First, please download the following file.
> 
> https://github.com/doergestim/Revamped-Android-Labs/blob/main/tools/final.apk

## What is an intent?

In this lab we will be using "intents" to bypass access controls.

An "intent" is a "message object" typically used communicate between different activities in your application.

However, sometimes intents can be sent between applications. A legitimate example of this might be your camera accepting an intent from your banking app in order to take a photo of a check.

So in order to analyze intents on our device, we need to create a [manifest file](https://www.blackhillsinfosec.com/field-guide-to-the-android-manifest-file/).

Next, we need to run the following command:

```bash
apktool d final.apk
```

This command will take the `final.apk` file and decompress it (since it is a zip) into multiple different directories.

![Apktool d command](/images/Screenshot%20From%202026-10-02%2010-59-38.png)

This allows us to go through and look at the `manifest.xml` file.

Once this process finishes, go ahead and navigate into the `final` directory and run `ls` to list the contents of the folder:

![ls command](/images/Screenshot%20From%202026-10-02%2011-00-26.png)
As you can see, we now have a file named `AndroidManifest.xml`. This is where we will be looking for the intents that we can access and manipulate.

Let's continue by running the following command:

```bash
less AndroidManifest.xml
```

Now we can press forward slash (`/`) to initiate a search of the output.

We want to search for activities, so let's type `activity` and hit `Enter`:

![less command](/images/Screenshot%20From%202026-10-02%2011-01-01.png)

If done correctly, you should see this:

![Searching for activity](/images/Screenshot%20From%202026-10-02%2011-01-30.png)

These are all of the exported activities. The following is an expansion of one of the entries to provide more context:

```
<activity
    android:name=".AccountDetails"
    android:parentActivityName=".AccountViewMain"
    android:exported="true">
        <meta-data
            android:name="android.app.lib_name"
            android:value="" />
</activity>
```

Things like searches, check deposits, etc., will all show up here, meaning that they are accessible to us.

This also means that the activity can be launched by a process outside of the application.

For our example, we will do this by using our best friend `adb`.

Before moving on, go ahead and press `Q` in the terminal to exit the `less` view.

## Invoking Intents from ADB

By using the adb shell, we can execute a specific activity within the app.

The following command executes the specified activity after the `-n` option.

`adb shell am start -n com.bhis.thehackerbank/.{ACTIVITY NAME}`

The name of each activity present in the application can be found in the [manifest file](https://www.blackhillsinfosec.com/field-guide-to-the-android-manifest-file/), but for this example, we will use the expanded entry from earlier:

![Expanded activity](/images/red2.png)

In the expanded entry, we see that the activity name is `.AccountDetails`, so we will substitute that into the above command and run it in our terminal:

```bash
adb shell am start -n com.bhis.thehackerbank/.{ACTIVITY NAME}
```

![Intent Command](/images/red3.png)

Shortly after making the command the app is starting our intent.

![Output of first Intent Command](/images/red4.png)

We can also pass data when starting intents with ADB. For example, running the below command results in a different result than running the command above.

```bash
adb shell am start -n com.bhis.thehackerbank/.AccountDetails --es "USER_COOKIE" "nothinginparticular"
```

![Second Intent Command](/images/Screenshot%20From%202026-10-02%2012-30-20.png)

When executed correctly, the app will behave slightly differently and give us even more information, and you should something similar to the following:

![Output of second intent command](/images/red5.png)

The second command requires us to know the name of the intent extra. These can be found by looking at the source code in a program such as `jadx`.

At this point, another thing to check is where you can go from here. With this application, pressing the back arrow will result in the application crashing.

However this is not always the case...

***Want to go back?  
[Previous Lab](https://github.com/doergestim/Revamped-Android-Labs/blob/main/Labs/LAB_06_Root_Detection_Bypass/instructions.md)***

***Looking for a different lab?  
[Lab Directory](https://github.com/doergestim/Revamped-Android-Labs/blob/main/navigation.md)***
