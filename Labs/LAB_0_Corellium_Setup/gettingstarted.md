# Setup Corellium and Establish Remote Connections
Corellium URL: https://app.corellium.com/login

I CANNOT STRESS THIS ENOUGH!!!!!!

REMEMBER TO POWER OFF YOUR VM WHEN NOT WORKING ON LABS!!!!

Corellium WILL CHARGE YOU!!!!!

## Creating an Android Device in Corellium
Login to your Corellium account and select the **DEVICES** tab and click **CREATE DEVICE**.

![](images/createdevice.png)


Next, select "Default Project."

![](images/defaultproject.png)

Next, select **ANDROID -> Generic Android** and select it.

![](images/selectandroidoption.png)

![](images/selectgenericandroid.png)


Select the firmware package, **12.0.0 (Build r26 userdebug)**, from the dropdown menu and click **SELECT**.

![](images/firmwareselect.png)

![](images/pressselect.png)

Leave the checkbox unchecked and select **CREATE DEVICE**

![](images/creatingdevice.png)

Corellium will then proceed to create the virtual deivce. 

![](images/progressbar.png)

Once the build is complete, you should have a virtual Android device with menu options as shown below.

![](images/completedsetup.png)

**Please note, if it does not say "Rooted" next to the device name, it is becasue you did not select userdebug firmware.  Please click the trash icon in the upper right, delete the device and start over from the beginning.**

***

## Navigating Menu Items in Corellium ##



The menu items associated with your Android device should look like the following.

![Device Menu Items](images/device-menu-options.jpg)

The following list contains a high-level description of each menu item.
 * **Connect** - This feature allows the tester to remotely connect to the virtual Android device for testing. 
 * **Files** – Corellium gives you control over the device filesystem, allowing you to upload, download, delete, modify, and search for files.
 * **Apps** – Manage Apps installed on the virtual device (i.e., install/uninstall, launch/kill apps).
 * **Network** – The Network Monitor captures, presents, and monitors HTTPS traffic, transparently defeating certificate pinning.  
 * **CoreTrace** - Ability to trace system calls for dynamic analysis and reverse engineering which offers a quick way to understand a program’s behavior. 
 * **Settings** - Configuration settings for your virtual device, such as: number of CPU cores, allocated RAM, as well as custom boot and kernel options.  
 * **Frida** - Corellium's bulit-in Frida server to support process hooking and code injection for bypassing controls and/or reverse engineering purposes.
 * **Console** – Use the Console option to see system and kernel logs and quickly run commands without needing to connect over ADB or SSH.
 * **Sensors** - Configure your virual device's peripherals and sensors, such as: Battery status, Camera/Microphone, GPS coordinates, and Motion/Position/Orientation.
 * **Snapshots** - Save, clone, or restore your device's virtual state. 

## Remotely Connecting to your Virtual Mobile Device ##

Establishing a network connection between your mobile device and your testing platform is required to perform actions such as proxying and intercepting Internet traffic, which enables security practitioners to further evaluate a given mobile application under an active/running state. Additionally, tools like the Android Debugger (adb) can be leveraged remotely from the tester’s virtual machine to the virtual mobile device running in Corellium. 
In this section we will cover two options for establishing remote network capabilities to/from your MobileApp Virtual Machine and the virtual mobile device running in Corellium: SSH and VPN.

### SSH ###
1. Create a unique SSH keypair (public and private certificates) from your MobileApp VM:
    - From the command prompt, type the following command and hit enter.
    `ssh-keygen -t ed25519`
    - For the file name, enter “sshKey”, then hit enter.
    - Hit enter two more times to create the key with no passphrase.
    - The keypair should now be in your CWD (current working directory – to confirm, type the command ‘ls’ and you should see two files
        - sshKey – this is your private key and should remain on your MobileApp VM.
        - sshKey.pub – this is your public certificate, the contents of which will need to be added to your Corellium instance.

![ssh keypair](images/ssh-keypair.jpg)

2. Type the following command to display the contents of sshKey.pub to stdout of your terminal.

`cat sshKey.pub`

![ssh public key](images/sshKey.pub-contents.jpg)

3. From your Corellium account select "Connect". Then select Admin page at the bottom red box of the "Quick Connect" section.
   
<img width="920" alt="Screenshot 2023-12-30 at 11 10 20 AM" src="https://github.com/deruke/AndroidLabs/assets/22796374/de07456b-3ade-4086-9be1-eff60e16ca4e">


5. Under the **Authorized Keys** section, click **NEW KEY**, then select **SSH** for the *Key Type*, and copy the contents of your ssh public key (step 2 above) and paste it in the text box here. Click **CREATE**.

![Adding Authorized Key to Corellium](images/adding-public-key.jpg)

**NOTE:** Be sure to add the entire contents of your public key - starting with *"ssh-ed25519..."* and ending with your hostname *"...mobileapp@mobileapp-vm"*. The full content of your SSH key will be different.

#### Test the SSH Connection ####

1. Ensure that your Corellium virtual device is powered on.

![Device On](images/devices-on.jpg)

2. Navigate to the **Connect** menu item and copy the first command listed under **Quick Connect**.

![Connect - SSH](images/connect-ssh.jpg)

3. Return to your MobileApp VM, then paste the command into a terminal session. The command will be slightly different for each student; however, be sure to add the `-i sshKey` at the end of the command you just pasted.

![Connect - SSH Command](images/ssh-connect-command.jpg)

4. The above command will create an SSH tunnel between your MobileApp VM and the virtual device in Corellium. The command also sets a *control socket* and binds the connection to `localhost:5001`. Now we can connect the Android Debug Bridge (adb) using the following command.

`adb connect localhost:5001`

![ADB Connection](images/adb-commands-1.jpg)

5. With `adb` connected, we can run commands such as: 

`adb devices` - List of devices attached

`adb shell` - Shell acess to the Android device

![ADB Commands](images/adb-commands-2.jpg)

**NOTE:** There's a dedicated [lab](https://github.com/deruke/AndroidLabs/blob/main/Labs/adb/adb_cheatsheet/adb_cheatsheet.md) on utilizing adb and adb commands.

6. Finally, to terminate the SSH tunnel run the following command.

`ssh -Ssock -O exit proxy.corellium.com`

![Terminate SSH Tunnel](images/ssh-terminate.jpg)


### VPN ###

An alternative approach to SSH for remote connectivity is to use a VPN. Corellium offers support for VPN connections via an OpenVPN configuration which is obtained via the Corellium web UI. The Corellium VPN enables us to remotely route network traffic from the virtual mobile device, through our MobileApp VM. From the MobileApp VM, we can then perform network traffic inspection and intercept/manipulate the traffic using tools like Burp Suite.

1.	To begin, navigate back to the Corellium web UI and access your virtual mobile device. Then click on the **Connect** tab.

**NOTE**: Ensure that your virtual device is powered on.

2. Scroll down to the **Connect via VPN** section and click **DOWNLOAD OVPN FILE**.

![Download OVPN File](images/download-openvpn.jpg)

3. Save the downloaded OVPN file to a directory/location on the MobileApp VM.
<img width="582" alt="Screenshot 2023-12-30 at 11 20 51 AM" src="https://github.com/deruke/AndroidLabs/assets/22796374/4d376964-ea1f-4e00-9e28-5e6581e4b87e">

![Downloaded OVPN File](images/openvpn-local.jpg)

4. Prior to establishing the VPN, run the following command from a terminal session on your MobileApp VM to get a current list of network interfaces.

`ip a`

 - The following screen capture provides an example of the list of network interfaces.

 ![List of Interfaces](images/list-interfaces.jpg)

**NOTE**: Your output may not match exactly – the key takeaway here is understanding the current network interfaces prior to establishing the VPN.

5. Next, run the following command from a terminal session on your MobileApp VM.

`sudo openvpn ~/Downloads/corellium.com\ VPN\ -\ Default\ Project.ovpn`

 - We can see from the below output that OpenVPN created a new network interface, **tap0**, with the IP address: **10.11.3.2**

![tap0 interface](images/tap0-interface.jpg)

**NOTE**: The IP address assigned to the tap0 interface may be different. <ins>Additionally, each time the VPN is established there is a possibility that the assigned IP may change</ins>.

6. Run the `ip` command once again from a different terminal to see the tap0 interface and the currently assigned IP address.

`ip a`

![tap0 interface assigned IP](images/tap0-interface-w-IP.jpg)

7. Test the VPN connection by navigating back to Corellium’s web UI and selecting the **Console** tab. Then enter the following command in the console shell.

`ping <tap0 assigned IP>`

![corellium console - ping tap interface](images/corellium-console-ping.jpg)

8. While the ping command is running in the console session, return to the MobileApp VM and execute the following *tcpdump* command. 
`sudo tcpdump -nni tap0`

 - In the screen capture below, we can see that the virtual mobile device (10.11.0.3) and our MobileApp VM (10.11.3.2) are communicating over a private network connection.

 ![tcpdump attached to tap0 interface](images/tcpdump-tap0.jpg)
