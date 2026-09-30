# Setup Burp Suite for Proxy and Interception Testing 

>[!WARNING]
>
> I CANNOT STRESS THIS ENOUGH!!!!!!
>
>REMEMBER TO <b style="color:#FF0000;">POWER OFF YOUR CORELLIUM DEVICE</b> WHEN NOT WORKING ON LABS!!!!
>
><b style="color:#FF0000;">CORELLIUM WILL CHARGE YOU!!!!!</b>

In this lab we will set up a web proxy using *Burp Suite's Community Edition* and route traffic from our virtual mobile device in Corellium to our MobileApp VM where Burp resides. Following this setup, we will have the capability to intercept web requests made by a given mobile application, as well as inspecting and/or manipulating network traffic for the purposes of mobile application testing. 

>[!IMPORTANT]
>In order to ensure network traffic is routed from the virtual mobile device to our MobileApp VM, we first need to establish a VPN connection between the two hosts.<br>
>More details on how to setup a VPN connection can be found [here](/Labs/LAB_00_Corellium_Setup/gettingstarted.md/#vpn).

## Open and Configure Burp
1. With a VPN connection established, return to your MobileApp VM and launch Burp Suite Community Edition by clicking on the Burp icon in the Favorites Toolbar.

![](/Labs/LAB_04_Burp_Proxy_Setup/images/launchburp.png)

2. Once open, select **Temporary project**, then click **Next**

 ![](images/tempthennext.png)
 
 Then make sure **Use Burp Defaults** is selected, and hit **Start Burp**

 ![](images/defaultstartburp.png)

3. Once the project has started, use Burp's menu to navigate to **Proxy -> Options**

![](images/proxythenoptions.png)

4. Uncheck the "Running" checkbox for interface 127.0.0.1:8080

![](images/uncheckbox.png)

5. Next click **Add**, then **Bind to address -> Specific address**, and select the IP address assigned to the **tap0** interface. Also, enter the port number in the **Bind to port** field. 
When finished click **OK**.   

 ![](images/step5.png)

 >[!Note]
 >The address assigned to your **tap0** interface may be different. To ensure you select the correct IP for Burp to bind to, run the following command from a terminal session on your MobileApp VM.
 
 <pre>ip a show tap0</pre>
 
 ![tap0 interface](images/tap0-interface.jpg)

 6. You should now have an active listener in Burp.

 ![](images/activelistener.png)

You now have Burp's proxy set up and listening for incoming connections. In the next section of this lab, we will walk through the configuration of the virtual mobile device in Corellium.

## Configuring the Virtual Mobile Device's Proxy Settings

1. Navigate back to your Corellium device. We need to navigate to **Settings -> Network & internet** on our virtual device. To do this, start by clicking on the device screen. Then click and drag down from the bottom of the device's screen:

![](images/clickanddragup.png)

Now hit the settings icon:

![](images/clicksettings.png)

Next, click **Network & internet**:

![](images/networksettings.png)

2. Select **Internet**

![](images/selectinternetsettings.png)

3. Click on the *gear* icon of the *T-Mobile* connection.

![](images/tmobilesettings.png)

4. Under the settings for the *T-Mobile* interface, scroll down and select **Access Point Names**

![](images/selectaccesspointnames.png)

5. Select the **T-Mobile US** APN.

![](images/selecttmobileus.png)

6. Click on the **Proxy** and **Port** fields and enter the value matching Burp's proxy settings. In our instance, we will set **Proxy** to `10.11.3.2` and the **Port** to `8888`. Then select the *Kebab* icon in the top-right corner and click **Save**.

![](images/setproxyandport.png)


>[!IMPORTANT]
>If you don't save the proxy settings you will need to repeat the previous steps.

7. Navigate back to **Settings -> Network & internet -> Internet** and select the icon at the top-right corner to reset the network interface. This will reset the virtual device's network interface which enables the proxy settings to be recognized by the device.

![](images/resetinternet.png)

## Configure the Virtual Mobile Device's Certificate Trust for the Burp Proxy Certificate Authority (CA) - User-Trust

The following steps will walk you through the installation of Burp's CA certificate to the User-Trust Store on your Corellium virtual mobile device.

>[!IMPORTANT]
>Starting with Nougat (Android 7.0 - API level 24) certificates installed to the User-Trust store are ignored by default. However, with Corellium's implementation of Android devices, some native applications have been "patched" to trust the user cert store. If you are using a different mobile device solution for testing, Android devices 7.0+ (API >= 24) will require the Burp CA cert to be installed to the System-Trust store (see [Lab 5](/Labs/LAB_05_Import_Burp_Certificate_To_System_Trust_Store/burp-proxy-system-cert-trust.md) for adding Burp's CA to the System-Trust on Android devices).  

1. Return to your instance of Burp running on the MobileApp VM and navigate to: **Proxy -> Options** then click on **Import / export CA certificate**.

![](images/importcertificate.png)

2. Select the **Certificate in DER format** and then click **Next**.

![](images/certificateinder.png)

3. Now, hit **Select File...** and navigate into the **Downloads** folder. Then we need to name the file. In our case, we named it `BurpCA.cer`.

![](images/exportcert.png)

![](images/downloadsandnamefile.png)

Now hit **Save**. You should see something similar to the following:

![](images/savenextcert.png)

Finally, hit **Next** and then **Close**.

>[!IMPORTANT]
> If you are not running your Corellium device *INSIDE* the MobileApp VM, you will need to move the exported certificate file on to your local device **BEFORE CONTINUING**
>
> To do this, navigate to where the certificate is saved, and copy and paste it onto your local system in order to upload it in the next step.

4. Go back to your Corellium instance and click **Files** in the navigation menu. Navigate to the `/mnt/sdcard/Download` directory by typing in the search bar. Then, click upload and select the `BurpCA.cer` file that you just exported.

![](images/uploadfile.png)

5. Return to the virtual mobile device's home screen (in Corellium) and select the **Settings** icon using the same "Swipe Up" technique from earlier.

![Settings](images/clicksettings.png)

6. Scroll down and select **Security** then find **Encryption & credentials** and select it.

![](images/selectsecurity.png)

![](images/hitencryptionandcredentials.png)

7. Under **Encryption & credentials** click on **Install a certificate**, then click **CA certificate**

![](images/clickinstallcertificate.png)

![](images/cacertificate.png)

8. A prompt will warn you of the dangers involved with installing a CA certificate...click **INSTALL ANYWAY** to proceed.

![](images/installanyway.png)

9. Next, click the *Hamburger* icon and select **Downloads**, then click on the Burp certificate you uploaded in step 4.

![](images/hamburgericon.png)

![](images/selectthecertificate.png)

10. If successful, a temporary pop-up wil appear indicating *"CA certificate installed"*.

![](images/successfulinstallmessage.png)

11. To verify that the certificate was installed to the User-Trust store, navigate to **Settings -> Security -> Encryption & credentials -> Trusted credentials -> User**

![](images/trustedcredentials.png)

![](images/showtrusteduser.png)

12. Launch the Web View app from the virtual device in Corellium and enter a common Internet resource, such as *https://www.blackhillsinfosec.com*.

![](images/openwebview.png)

![](images/navigatetoasite.png)

13. Finally, navigate back to your MobileApp VM and from within Burp, navigate to **Proxy -> HTTP history**. 

You should see your web traffic processed by Burp's proxy.

![](images/httphistoryburp.png)

### Not seeing traffic in Burp?

Use the following steps to troubleshoot if necessary:

1. Ensure the VPN connection between Corellium and the MobileApp VM is established.

2. Ensure the Burp proxy options (IP Address and Port) are properly set and the proxy is listening.

3. Check the User-Trust store to ensure the CA certificate is installed.

4. Check the proxy settings on the virtual mobile device to ensure they match that of the **tap0** interface as well as Burp's proxy settings.

5. If everything checks out and still no luck:
    - Reboot the virtual device
    - Cycle the VPN connection (off/on again)
    - If the **tap0** interface receives a different IP than what was previously set in Burp: update the settings in Burp as well as the proxy settings on the device.
    - Be sure to save the proxy settings on the virtual device 

***                                                                 

<b><i>Continuing the course? </br>[Next Lab](/Labs/LAB_05_Import_Burp_Certificate_To_System_Trust_Store/burp-proxy-system-cert-trust.md)</i></b>

<b><i>Want to go back? </br>[Previous Lab](/Labs/LAB_03_Static_Analysis/instructions.md)</i></b>

<b><i>Looking for a different lab? </br>[Lab Directory](/navigation.md)</i></b>


