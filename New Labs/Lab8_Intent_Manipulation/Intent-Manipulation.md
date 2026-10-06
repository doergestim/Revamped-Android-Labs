# Intent Manipulation

If your device isn't booted yet, launch it with the following, look at [Lab Setup](/New%20Labs/NewLab_Setup.md/#Launching%the%Device)

## Setup



### Prerequisites

You will need to add jre so the apktool can work. Do it with this command in terminal.

```bash
sudo apt update
sudo apt install -y default-jre
```

### Installing apktool

Apktool is a software used for decompiling and building again apk files.



> [!Note]
> 
> decompiling - taking the file that is ready to be put on the phone as an application and stripping it into file

The quickest way of installing it is on their official website:

![Linux instaltion process](./images/setup_photo.png)

First of all we visit their website:

https://apktool.org/docs/install

Then we click the `linux wrapper script` with right click, save link as apktool.

![Linux wrapper script](./images/setup_01.png)

![Linux wrapper script downloaded](./images/apktool_script_on_desktop.png)

Next we click the green link that will redirect us to downloading page. Then we pick the first option of apktool and we click it.

![Installing proccess](./images/Installing.png)

![Installing proccess further](./images/Installing_further.png)

Now we are renaming the apktool that we just downloaded. 

![Renaming the jar file](./images/renaming.png)

>[!NOTE]
> We can use `mv` just to rename the file. First name of a file after `mv` is the name of a file that we want to change and the second one is the desired name of the file.

Following step require us to move both of the files into ```/usr/local/bin``` folder. We can do it with mv command.

``` bash
sudo mv apktool.jar /usr/local/bin 
```
``` bash
sudo mv apktool /usr/local/bin
```
>[!NOTE]
>`Mv` command is used for moving files as well. First after the command is the file name and the second is the desired file destination. In order to move those files into this certain directory we need `super user privilages`. This is why we use `sudo` command

Now it is the time that we make both of the files executable. What it means is that we give a file execute permission, meaning the system is allowed to run it as a program or script.

``` bash
chmod +x /usr/local/bin/apktool
```

``` bash
chmod +x /usr/local/bin/apktool.jar
```
![Giving executive permissions](./images/Exe_permissions.png)

Now we try to open apktool in CLI (terminal).

![Opening apktool](./images/opening_apktool.png)

This is how you should see it.


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

![Apktool d command](./images/img01.png)

This allows us to go through and look at the `manifest.xml` file.

Once this process finishes, go ahead and navigate into the `final` directory and run `ls` to list the contents of the folder:

![ls command](./images/img02.png)
As you can see, we now have a file named `AndroidManifest.xml`. This is where we will be looking for the intents that we can access and manipulate.

Let's continue by running the following command:

```bash
less AndroidManifest.xml
```

Now we can press forward slash (`/`) to initiate a search of the output.

We want to search for activities, so let's type `activity` and hit `Enter`:

![less command](./images/img03.png)

If done correctly, you should see this:

![Searching for activity](./images/img04.png)

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

![Expanded activity](./images/img05.png)

In the expanded entry, we see that the activity name is `.AccountDetails`, so we will substitute that into the above command and run it in our terminal:

```bash
adb shell am start -n com.bhis.thehackerbank/.{ACTIVITY NAME}
```

![Intent Command](./images/img06.png)

Shortly after making the command the app is starting our intent.

![Output of first Intent Command](./images/img07.png)

We can also pass data when starting intents with ADB. For example, running the below command results in a different result than running the command above.

```bash
adb shell am start -n com.bhis.thehackerbank/.AccountDetails --es "USER_COOKIE" "nothinginparticular"
```

![Second Intent Command](./images/img08.png)

When executed correctly, the app will behave slightly differently and give us even more information, and you should something similar to the following:

![Output of second intent command](./images/img09.png)

The second command requires us to know the name of the intent extra. These can be found by looking at the source code in a program such as `jadx`.

At this point, another thing to check is where you can go from here. With this application, pressing the back arrow will result in the application crashing.

However this is not always the case...

***Want to go back?  
[Previous Lab](https://github.com/doergestim/Revamped-Android-Labs/blob/main/Labs/LAB_06_Root_Detection_Bypass/instructions.md)***

***Looking for a different lab?  
[Lab Directory](https://github.com/doergestim/Revamped-Android-Labs/blob/main/navigation.md)***
