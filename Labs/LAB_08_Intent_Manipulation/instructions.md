# Intent Manipulation #

>[!WARNING]
>
> I CANNOT STRESS THIS ENOUGH!!!!!!
>
>REMEMBER TO <b style="color:#FF0000;">POWER OFF YOUR CORELLIUM DEVICE</b> WHEN NOT WORKING ON LABS!!!!
>
><b style="color:#FF0000;">CORELLIUM WILL CHARGE YOU!!!!!</b>

## What is an intent? ##

In this lab we will be using "intents" to bypass access controls.

An "intent" is a "message object" typically used communicate between different activities in your application.

However, sometimes intents can be sent between applications. A legitimate example of this might be your camera accepting an intent from your banking app in order to take a photo of a check.

So in order to analyze intents on our device, we need to create a <a href="https://www.blackhillsinfosec.com/field-guide-to-the-android-manifest-file/">manifest file</a>.  

>[!Note] 
>Before we begin, make sure that your VPN connection is established, and that your device is connected.

In a terminal on our MobileApp VM, navigate to the directory that has the `final.apk` file that we downloaded in Lab 6. In our case, that will be the `Downloads` directory.

Next, we need to run the following command:

<pre>apktool d final.apk</pre>

![](images/runapktooldfinal.png)

This command will take the `final.apk` file and decompress it (since it is a zip) into multiple different directories. 

This allows us to go through and look at the `manifest.xml` file.

Once this process finishes, go ahead and navigate into the `final` directory and run `ls` to list the contents of the folder:

![](images/androidmanifestxml.png)

As you can see, we now have a file named `AndroidManifest.xml`. This is where we will be looking for the intents that we can access and manipulate.

Let's continue by running the following command:

<pre>less AndroidManifest.xml</pre>

Now we can press forward slash (`/`) to initiate a search of the output. 

We want to search for activities, so let's type `activity` and hit `Enter`:

![](images/searchless.png)

If done correctly, you should see this:

![](images/searchresult.png)

These are all of the exported activities. The following is an expansion of one of the entries to provide more context:

```xml
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

## Invoking Intents from ADB ##

By using the adb shell, we can execute a specific activity within the app.

The following command executes the specified activity after the `-n` option.

<pre>adb shell am start -n com.bhis.thehackerbank/.{ACTIVITY NAME}</pre>

The name of each activity present in the application can be found in the <a href="https://www.blackhillsinfosec.com/field-guide-to-the-android-manifest-file/">manifest file</a>.

![Redacted](images/redacted.png)

`adb shell am start -n com.bhis.thehackerbank/.AccountDetails`

We can also pass data when starting intents with ADB. For example, running the below command results in a different result than running the command above.

`adb shell am start -n com.bhis.thehackerbank/.AccountDetails --es "USER_COOKIE" "nothinginparticular"`

When executed correctly, the app will behave slightly differently.<br>
![Redacted v2](images/redacted2.png)

The command above requires us to know the name of the intent extra. These can be found by looking at the source code in a program such as jadx.

Another thing to check at this point is where you can go from here. With this application, pressing the back arrow will result in the application crashing, however this is not always the case.
