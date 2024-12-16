<img align="right" src="https://raw.githubusercontent.com/graphiks/woa-raphael/main/media/raphael.png" width="350" alt="Windows 11 Running On raphael">


# Running Windows on the  Mi 9T Pro / Redmi K20 Pro 

## Installing Windows

### Prerequisites

- [Windows on ARM image](https://worproject.com/esd)
- [UEFI image](https://github.com/woa-raphael/woa-raphael/releases/download/raphael-uefi/xiaomi-raphael_NOSB.img)
- [Drivers](https://github.com/woa-raphael/raphael-drivers/releases/tag/1.1)
- [Msc script](https://github.com/graphiks/woa-raphael/releases/download/raphael-partitioning/msc.sh)
- [TWRP](https://github.com/graphiks/woa-raphael/releases/download/raphael-partitioning/twrp.img) (should already be installed)

##### Boot to TWRP
> If rebooting on the last page has replaced your recovery back to stock, flash it again in fastboot with:
```cmd
fastboot flash recovery path\to\twrp.img reboot recovery
```

##### Running msc
> Put msc.sh in the platform-tools folder, then run:
```cmd
adb push msc.sh / && adb shell sh msc.sh
```

## Diskpart
>  [!WARNING]
> DO NOT ERASE ANY PARTITION WHILE IN DISKPART!!!! THIS WILL ERASE ALL OF YOUR UFS!!!! THIS MEANS THAT YOUR DEVICE WILL BE PERMANENTLY BRICKED WITH NO SOLUTION! (except for taking the device to Xiaomi or flashing it with EDL, both of which will likely cost money)

After you hear your phone getting reconnected to your PC run:
```cmd
diskpart
```
> Run the following commands in the newly opened window

##### Finding your phone
```cmd
lis dis
```
> This will show all available disks. Find the disk number of your phone and replace it with "$" in the command below

> (it will probably show as 0 B free)
```cmd
sel dis $
```

##### Selecting the ESP partition
```cmd
lis par
```
> This will print out all of the partitions in the selected disk. Check if they match up with your device and replace "$" with the number of the ESP partition (usually 32)
```cmd
sel par $
```

##### Formatting the ESP partition
> This will format ESP to FAT32
```cmd
format quick fs=fat32 label="System"
```
> Now add letter Y to the ESP partition
```cmd
assign letter y
```

##### Selecting the Windows partitiom
> Replace "$" in the command below with the number of the Windows partition, usually 33. If you don't know the number, run "lis par" again
```cmd
sel par $
```

##### Formatting the Windows partition
> This will format Windows to NTFS
```cmd
format quick fs=ntfs label=Windows
```
> Now add letter X to the Windows partition
```cmd
assign letter x
```

##### Exit diskpart
```cmd
exit
```

## Installing Windows
> Replace `path\to\install.esd` with the actual path to install.esd.
```cmd
dism /apply-image /ImageFile:path\to\install.esd /index:6 /ApplyDir:X:\
```

##### Installing Drivers


```cmd
DriverUpdater.exe -p X: -d .\definitions\Desktop\ARM64\Internal\raphael.xml -r .
```
  
##### Create Windows bootloader files
```cmd
bcdboot X:\Windows /s Y: /f UEFI
```

###### Configuring bootloader files
> Run these 4 commands seperately
```cmd
cd Y:\EFI\Microsoft\Boot
```
```cmd
Y:
```
```cmd
bcdedit /store BCD /set "{default}" testsigning on
```
```cmd
bcdedit /store BCD /set "{default}" nointegritychecks on
```
```cmd
bcdedit /store BCD /set "{default}" recoveryenabled no
```

## Unassign disk letters
> So that they don't stay there after disconnecting the device
```cmd
diskpart
```

##### Select the Windows volume of the phone
> Use `lis vol` to find it, it's the one named "Windows"
```diskpart
select volume <number>
```
##### Unassign the letter X
```diskpart
remove letter x
```

##### Select the ESP volume of the phone
> Use `list volume` to find it, it's the one named "ESP"
```diskpart
select volume <number>
```
##### Unassign the letter Y
```diskpart
remove letter y
```

##### Exit diskpart
```diskpart
exit
```

## Backing up boot images

##### Reboot your recovery
> To remove the msc script
- Reboot to recovery through TWRP, or run
```cmd
adb reboot recovery
```

##### Push the UEFI to your phone
> Drag and drop the UEFI (xiaomi-raphael_NOSB.img) to your phone

##### Back up your Android boot image
Use the TWRP backup feature to backup your Android boot image. Name this backup "Android"

##### Flash the UEFI
Use the TWRP install feature to flash the UEFI image to your boot partition. Select "install image", then locate the image.

##### Back up your Windows boot image
Use the TWRP backup feature to backup your Windows boot image. Name this backup "Windows"

##### Boot into Windows
After having flashed the UEFI image, reboot your phone.

Your device will now set up Windows. This will take some time. It will eventually reboot, and after that the initial setup (oobe) should launch.

## Setting up Windows
> [!IMPORTANT]
> USB will not work except if you have a powered USB hub. We can fix this after we get into the desktop.

Before continuing with setup, open the accessibility menu in the bottom right corner and enable the on-screen keyboard, then tap FN+SHIFT + F10 (if it asks you to tap somewhere to type just tap the background) which will open a command prompt, in which you will need to type:
```cmd
cd oobe
```
After that, type:
```cmd
bypassnro.cmd
```
> This command will skip the Microsoft Account requirement.
> 
Your device will now reboot. Continue setup as normal. Make sure to press the "I don't have internet" button when you reach the **Let's connect you to a network** section.

It is recommened to also read the [post install guide](postinstall.md).


## [Next step: Setting up dualboot](/guide/dualboot.md)
