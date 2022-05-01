---
layout: post
title:  "Setting up a Basic Android Instrumentation with Frida!"
date:   2022-04-30 20:13:58 +0300
categories: Android Pentesting
---
Frida is a powerful dynamic instrumentation toolkit for pen testers, developers, reverse-engineers and security researchers. It’s is scriptable, implying you can write custom scripts to autonomously perform functions such as:
* Hook into functions.
Take a peek into the code of private apps
* Inject code to modify output
* Bypass root
* SSL pinning bypass
* Fingerprint bypass
* Bypassing different software-side security locks.
It has an array of tools that span a wide range of test environment. And did I mention its free!! 

Okay, enough blabbing. Lets get straight into how to setup Frida and run a basic first Android testing project.

**Requirements**

* Frida
* Genymotion
* Virtual-box
* Android platform tools
* Burpsuite

**Install Frida**

```css
    	$pip install frida
	$pip install frida-tools    
```

1. Go to [Frida releases](https://github.com/frida/frida/releases) to download the Frida version for our little test.
2. Search for the [frida-server-15.1.17-android-x86.xz tool](https://github.com/frida/frida/releases/download/15.1.17/frida-server-15.1.17-android-x86.xz).

This is important because we will be running the tool on an emulated environment on our Ubuntu desktop with an x86 architecture. 

If you were using a real mobile device for the test, you might want to download the ARM version of frida tools.

3. On a new terminal, navigate to where you downloaded the file and extract it.
```css
	$xz -d  frida-server-15.1.17-android-x86.xz
```
Rename it however you want (or don’t), I’ll name mine “frida-server”
```
	$mv frida-server-15.1.17-android-x86 frida-server
```
4. Go to your ADB path
```css
	$cd /opt/genymobile/genymotion/tools/
```
5. Push the file to the device
```css
	$./adb push /home/ash/tools/frida-server /data/local/tmp
```
6. Change the permission of the file:
```css
	$./adb shell "chmod 755 /data/local/tmp/frida-server"
```
7. Open a new adb shell
```css
	$./adb shell "/data/local/tmp/frida-server &"
```
Output should be as indicated below

8. Now on a new terminal in the /opt/genymobile/genymotion/tools/ path, run the following command to run frida:

To check if we have access to the files in the emulated device, enter the following command.
```css
	$frida-ps -U
```

**Generating the CA certificate**
1. Go to Burpsuite > proxy > options.
2. Import/export certificates
3. Export Burp CA- Select the Certificate in DER
4. Choose a save location name it and and download. i.e. `cacert`
5. Navigate to your download location and rename the certificate to have a `.der` extension. i.e. cacert.der

**Operations on Genymotion**
**Converting the Burp certificate from DER to PEM**

1. Open a new terminal
2. Navigate to the location in which you saved the `cacert.der` file.
3. Enter the following commands:
```css
    $openssl x509 -inform DER -in cacert.der -out cacert.pem; ls cacert.pem
```
The output should be: `cacert.pem`

4. Run this command next.
```
	$openssl x509 -inform PEM -subject_hash_old -in cacert.pem |head -1 |\
   xargs -I $ sh -c 'mv cacert.pem $.0; ls $.0'
```
The output should be: `9a5ba575.0`

5. Next step is to move the certificate to the device.

6. But first, we need to install Android Debugging Bridge:
```
	$sudo apt install adb
```
7. Run adb as root:
```	
    $adb root
```
You might get the following error. 

                **insert image**
If so, navigate to `/opt/genymobile/genymotion/tools/`
If you cant access the Genymotion directory that way, check where adb is installed: 
```	
    $which adb (not that this is the one we installed)
```
But we want to use the one that came with Genymobile (at least for those who got the error above)
Just make sure you are on the genymotion path when running the $adb root command.

Now retry running as root as follows:
```	
    $./adb root
```
Remount adb:
```	
    $./adb remount
```
Push the certificate file to the phone:
```
	$./adb push ~/Downloads/9a5ba575.0 /sdcard/
```
Move the file from the sdcard to the directory as follows:
```	
    $mv /sdcard/9a5ba575.0 /system/etc/security/cacerts
```
Change the permissions:
```	
    $chmod 644 systemetc/security/cacerts/9a5ba575.0

    Exit.
```
**Back to Genymotion**

Go to Settings> security> trusted credentials>PortSwigger

Everything should be working fine. Happy testing!!
