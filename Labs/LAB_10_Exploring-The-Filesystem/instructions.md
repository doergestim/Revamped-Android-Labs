# Exploring The Filesystem #

I CANNOT STRESS THIS ENOUGH!!!!!!

REMEMBER TO <b style="color:red;">POWER OFF YOUR CORELLIUM DEVICE</b> WHEN NOT WORKING ON LABS!!!!

<b style="color:red;">CORELLIUM WILL CHARGE YOU!!!!!</b>
***

In this lab, we are going to investigate the file system of the application TheHackerBank.

**Note:** This is most effective after you have browsed through the functionality of the app, since it is likely that things get written to the filesystem during runtime.

If you cd to the applications internal storage and run an `ls -la` command you should see the following:
![screenshot](images/fs.png)

We will take the files off of the device for offline analysis. There are several ways this can be done:

One option is to make a backup of the app. We will be able to do this only if allowbackup="true" in the manifest file.

`adb backup -f Backup.ab  com.bhis.thehackerbank`
You will need to click "BACK UP MY DATA" on the phones gui.
![](images/backup.png)

Another method download all of the applications data is with the adb pull command.

`adb pull /data/data/com.bhis.thehackerbank`

**Note:** These two methods produce the same result, however, adb pull will only work if the device is rooted, and adb backup will only work if the app allows backups, but will work regardless if the device is rooted.

If there is a large amount of data it may be beneficial to move the data to a tar archive on the system before pulling it down.

`tar cvzf filesystem.tar.gz /data/data/<package-name>`

Then on the host you are using for analysis run `tar -xvzf filesystem.tar.gz`
After that you're on your won for analysis.

For example: `grep -InrE pass|key|token|api|cred|auth|cookie` might be a good place to start.

additionally, the application may use external storage. You might have noticed The Hacker Bank does not have the WRITE_EXTERNAL_STORAGE permission declared in the manifest file. The app does however write to external storage. Since android 10, android uses a "Scoped Storage" model. Can you figure out where those "Check Deposit" images go?

## Extra Credit ##
You have root on the device and can browse data data for any of the applications. Look through the file system for another app of your choice.
