# rootlessjb3-guide
Guide on how to install apps / tweaks on RootlessJB3

A few days ago, rootlessjb3 was released and introduced the possibility of finally installing jailbroken tweaks/apps agin.
There has been a few guides floating around the internet on how to do this, but many of the guides aren't detailed enough.
Hopefully this will shed some light on the problems you may encounter when trying to do this yourself.

## Installing Tweaks / Apps:
#### The following is taken place on OS X.

Requirements:
  - ldid2
  - .deb file
  - patcherplus (https://github.com/M4cs/rootlessJB-Patcher/releases/download/1.0.1/patcherplus)
  - dpkg-deb installed ('brew install dpkg')
  
First step, is to make sure you actually have the .deb file.
If you don't know what a .deb file is, it's basically a zipped up file containing the contents of the tweak/app.

1:  Folder setup
Head over to your Desktop and create a folder for your tweak and lay it out like so:

  - mytweak
    - mytweak.deb
    - patcherplus
    
2:  Modifying the patcherplus permissions
Now open terminal, change to your "mytweak" directory and modify the permissions of patcherplus to +x.

Terminal:
```
cd ~/Desktop/mytweak
chmod +x patcherplus
```

3:  Running patcherplus
What patcherplus will do, is run through the .deb file, and sign the .dylib files for MobileSubstrate with the correct entitlements. NOTE if you're planning on installing an .app file from the .deb, you will NEED to sign this manually with ldid2. patcherplus requires you to enter the .deb file, then the output file after, so enter the following in terminal:

Terminal:
```
patcherplus
mytweak.deb
mytweakoutput
```

4:  Check contents
Once patcherplus has finished running, you will be left with a new folder (in our case "mytweakoutput"). This folder contains the contents of the extracted .deb file; with the .dylib files signed (inside Library/MobileSubstrate/DynamicLibraries). The .dylib files are used by mobilesubstrate for tweaks. NOTE: ONLY the dylib files get signed.. if you're installing a .app file, follow the optional step below.

5:  (FOR .app installations only)
If your output folder contains a folder called "Applications", you will need to follow this step.
The executable in the .app file needs to get signed with the proper entitlements.. this is a MUST.
The entitlements must AT LEAST include these 2 to function. (Obviously they can include more, but these are minimum).

```
<key>platform-application</key>
<true/>
<key>com.apple.private.security.container-required</key>
<false/>
```

We're going to open terminal now and temporarily export our .ent file to the Desktop.
To do this, we can output the entitlements to the terminal like so:

Terminal:
```
ldid2 -e Applications/myapp.app/myapp
```

This will output contents that look like an XML file, which you will need to copy and paste into a new file (on your desktop in this case) - call it myapp.ent

Now we need to resign the executable with this .ent file. So open terminal and enter the following (yes, there's no space between the arguments):
```
cd ~/Desktop
ldid2 -Smyapp.ent
```

The executable will now be signed with the valid entitlements.

.app files:
Before copying your .app file to your ios device, first we need to check if CydiaSubtrate.framework is referenced anywhere. Do to this, use otool:

```
otool -L myapp.app/myapp
```

If you see see CydiaSubstrate.framework in there, you will need to swap the references over.
Do to this use "install_name_tool (you will need to get a copy of CydiaSubstrate.framework and copy it over to /var/LIBS/Frameworks too)":

```
install_name_tool -change /old/directory/CydiaSubstrate.framework/CydiaSubstrate /var/LIB/Frameworks/CydiaSubstrate/CydiaSubstrate myapp.app/myapp
```

6:  Copying the files to your iOS Device
Now we need to get these files over to your device, to do this, i recommend using a program like iMazing or iFunxBox.
I'm using iFunBox, on the left hand side, head over to the file system (/var/mobile/Media).. you should see other files like "AirFair", "DCIM", "iTunes_Control", etc. Create a folder called "Jailbreak", and copy your "mytweakoutput" folder to "Jailbreak".

7:  Moving the tweaks/apps to the appropriate places
Now the files are on your device, jailbreak your phone with rootlessjb3; this will allow you to ssh into your device from your mac. So open up terminal and ssh into your device (password is alpine), then change directory to /var/mobile/Media/Jailbreak:

Terminal:
```
ssh root@yourdeviceip
alpine
cd /var/mobile/Media/Jailbreak
```

Your mytweakoutput folder might look like this:

  - Applications
    - myapp.app
  - Library
    - MobileSubstrate
      - DynamicLibraries
        - tweakfile.dylib
        - tweakfile.plist
    - PreferenceBundles
      - mytweak.bundle
    - PreferenceLoader
      - Preferences
        - mytweak.plist
    - OPTIONAL
        
You need to copy these files to the appropriate locations (* means everything inside that folder).
      
  - Applications/myapp.app                      : /var/Apps
  - Library/MobileSubstrate/DynamicLibraries/*  : /var/LIB/MobileSubstrate/DynamicLibraries/
  - Library/PreferenceBundles/*                 : /var/LIB/PreferenceBundles/
  - Library/PreferenceLoader/Preferences/*      : /var/LIB/PreferenceLoader/Preferences
  - OPTIONAL                                    : /var/LIB

Now the files are in the correct places, you need to give your dylibs the correct permissions.
Open terminal and ssh into your device, then enter the following:

```
chown mobile:staff /var/LIB/MobileSubstrate/DynamicLibraries/mytweak.dylib
chmod 777 /var/LIB/MobileSubstrate/DynamicLibraries/mytweak.dylib
```

.dylib files need mobile:staff, and 777 permissions to run correctly.
If you have any preference bundles, they need to be updated with ONNLY 777 and not mobile:staff.

```
chmod -r 777 /var/LIB/PreferenceBundles/myapp.bundle
```

.app files.. DO NOT modify the permissions. If you've placed your .app file in /var/Apps.. you need to run uicache. This will install any .apps in /var/Apps for you automatically.

```
uicache
```

You should now see the app on your springboard (if you had a .app file).

8:  Reboot
You now need to hard-reboot your device.

9:  Re-jailbreak
Re-jailbreak your device with rootlessjb3 (make sure the tweaks button is ON).

10: Test
If you're expecting JUST a tweak and not an app, open Settings and check if your preference bundle is there.
If you're expecting an app.. there's 1 more step.. you need to allow amfi to let your app run, to do this go to terminal and type:

```
inject /var/containers/Bundle/Application/*/myapp.app/myapp
```

Check if the result is successul, if it's not.. you may need to go back a few steps and re-sign it with ldid2.
Done.
