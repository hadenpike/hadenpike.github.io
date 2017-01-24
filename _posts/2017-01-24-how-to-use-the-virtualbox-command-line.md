---
layout: post
title: How To Use the VirtualBox Command Line
---

# Introduction

[VirtualBox] is an application for creating virtual machines. A virtual
machine is a software based computer, running inside your own. They
have various uses, but one is to have an isolated environment in which
to test software. The
graphical user interface of VirtualBox has long been inaccessible to
the blind---leaving the command   line as the only option. Having said that, this guide may also be of interest to others. If
you are a system administrator, for example, you might use the
information here to automate the task of maintaining the virtual
machines your company uses. Technical manuals, such as the one for
[VBoxManage]---the command line interface to VirtualBox---do not
exactly make for scintillating reading, thus, this guide.

## Notes and Other Useful Bits of Information

+ You'll need to ensure that the path to the VBoxManage utility is
available at the command line. This means making sure it is in your
PATH environment variable. The steps to check and change this are
beyond the scope of this document.

+ Unless stated otherwise, all paths to files that you may enter must
be absolute not relative.

+ VM is an acronym for virtual machine and may be used at various
points throughout this guide.

+ Any text preceded by a dollar sign ($) indicates a command to be entered at the command line.

+ Finally, I'm using macOS, so any paths shown here will reflect that.

With that out of the way, let's begin.

# Creating A Virtual Machine

## Step 1: Finding the Operating System Identifier

The first step in creating a virtual machine is to know what operating
 system you're going to install. The operating system identifier is a
 string telling VirtualBox which OS the machine is going to run. These identifiers are pretty
 self-explanatory, and if you already know it, feel free to [skip to the next step.](#step-2-creating-the-virtual-machine)

Use the following command to list the operating system identifiers.

```
$ VBoxManage list ostypes
ID:          Other
Description: Other/Unknown
Family ID:   Other
Family Desc: Other
64 bit:      false

ID:          Other_64
Description: Other/Unknown (64-bit)
Family ID:   Other
Family Desc: Other
64 bit:      true 

ID:          Windows31
Description: Windows 3.1
Family ID:   Windows
Family Desc: Microsoft Windows
64 bit:      false
...
$
```

  Observe the output of this command, of which I've only shown a small portion. Look up the name of the Operating System in the table and take note of its ID.

## Step 2: Creating the Virtual Machine

We're going to create a 64-bit Debian virtual machine.

```
VBoxManage createvm
```

is the command to accomplish this. It accepts the following
options.

| Option | Description |
| ------ | ----------- |
| --name | The name of the virtual machine for humans. |
| --basefolder | If specified, instructs VirtualBox where to create the virtual machine files. |
| --ostype | The operating system identifier. We saw how to obtain this in [step 1](#step-1-finding-the-operating-system-identifier) (it's Debian_64 for the purposes of our example). |
| --register | Optionally performs the task of registering the VM with VirtualBox (see [step 3](#step-3-registering-the-machine) for details). |

The default location of virtual machines varies depending on your operating system and VirtualBox configuration, but should be something like:
  - Windows\: C\:\\Users\\username\\VirtualBoxVMS
  - MacOS\: $HOME/

I'm going to name this machine "Debian_64" and because I like to create
them on a dedicated drive, the basefolder parameter will be set to the
location of the drive.

```
$ VBoxManage createvm --name "Debian_64" --basefolder /Volumes/virtual_machines --ostype Debian_64
```

This command creates some configuration files and directories, but the important file is \<basefolder\>/\<vm_name\>/\<vm_name\>.vbox.

## Step 3: Registering the Machine

If you supplied the --register flag in [step 2](#step-2-creating-the-virtual-machine), you can proceed to [step 4](#step-4-creating-the-virtual-hard-disk). At this point, the basic configuration files that describe the virtual machine have been created. To procede, VirtualBox needs to know that your VM exists and where it's located. You can inform the system of the existance of your new virtual machine with the

```
VBoxManage registervm
```

command. You must supply the path to the .vbox file created previously along with the command. Thus, the command to register our sample machine is:

```
$ VBoxManage registervm /Volumes/virtual_machines/Debian_64/Debian_64.vbox
```

## Step 4: Creating the Virtual Hard Disk

Like the hard disk in a physical computer, the virtual hard disk is
where all of your data will be stored. We have a few different
options for what format of virtual drive we want, but we'll stick with
the .vdi format---the native format for VirtualBox.

Performing this task is done with the

```
VBoxManage createhd
```

command. We need to supply two options here.

| Option | Description | Allowed values |
| ------ | ----------- | -------------- |
| --filename | The name of the virtual hard disk file. | Anything you want---just remember it for later. This path can be relative. The filename extension must be included. |
| --size | The size of the virtual hard disk. | The size of the disk in megabytes[^1]. |

So to create a disk named "Debian_64_hd.vdi" of size 20480, the command would be:

```
$ VBoxManage createhd --filename "/Volumes/Virtual_Machines/Debian_64/Debian_64_hd.vdi" --size 20480
```

## Step 5: Adding the Storage Controller

The storage controller is what interfaces your storage and other media
to the machine. You can add one with the

```
VBoxManage storagectl
```

command. You must give it the name of your virtual machine, along
with the following.

| Option | Description | Allowed values |
| ------ | ----------- | -------------- |
| --name | The human-readable string identifying this storage controller. | Anything you like as long as you remember it. |
| --add | The storage controller type. | sata, ide, scsi, or floppy. |
| --hostiocache | Controls whether VirtualBox allows the host to cache disk files. | on or off[^2]. |
| --bootable | Specifies whether the machine can boot from the devices attached to this controller. | on or off |

Every VM must have at least one storage controller with the bootable flag set to on. Otherwise, the machine will not be able to boot any media.

We will create a storage controller of type sata, inventively named "SATA". As this is our first and only controller, we will set bootable to on.

```
$ VBoxManage storagectl "Debian_64" --name "SATA" --add sata --hostiocache on --bootable on
```

## Step 6: Adding Media to the Controller

Now we need to add our hard disk and any other media (such as that used for installation) to our virtual machine. We do this with the

```
VBoxManage storageattach
```

command. In addition to the VM name, we must provide the following.

| Option | Description | Allowed values |
| ------ | ----------- | -------------- |
| --storagectl | The name of the storage controller. | Name of an existing storage controller. |
| --device | The number of the ports device which is to be modified. | See the [VBoxManage] manual for details. |
| --port | The number of the storage controller's port which is to be modified. | See the [VBoxManage] manual for details. |
| --type | What kind of media this is. | dvddrive, hdd, fdd, or none. |
| --medium | The path to the media we want to attach. | Full path to media (.vdi file, .iso file, etc). |

To add our virtual hard disk to our sample machine:

```
$ VBoxManage storageattach "Debian_64" --storagectl "SATA" --port 0 --device 0 --type hdd --medium /Volumes/Virtual_Machines/Debian_64/Debian_64_hd.vdi
```

And for the Debian 64-bit installation media (your filename will very):

```
$ VBoxManage storageattach "Debian_64" --storagectl "SATA" --port 0 --device 1 --type dvddrive --medium /Volumes/Virtual_Machines/Debian_64/install_debian_64.iso
```

Note that we incremented the device number by one. This is done so we
don't detach our virtual hard disk and replace it with the
installation media.

## Step 7: Changing VM Settings

You have a fully configured virtual machine now, but the configuration
isn't optimal. You can change a setting (for example, how much memory
to allocate to the VM) with the

```
VBoxManage modifyvm
```

command. There are far to many settings to list here, but you can see
them all with the

```
VBoxManage showvminfo
```

command. For both, you need the VM name. Below is the output of running

```
$ VBoxManage showvminfo "Debian_64"
```

on my machine.

```
$ VBoxManage showvminfo "Debian_64"
Name:            Debian_64
Groups:          /
Guest OS:        Debian (64-bit)
UUID:            8f1954e7-4dc2-4af4-80b1-262faa45da74
Config file:     /Users/haden/Debian_64/Debian_64.vbox
Snapshot folder: /Users/haden/Debian_64/Snapshots
Log folder:      /Users/haden/Debian_64/Logs
Hardware UUID:   8f1954e7-4dc2-4af4-80b1-262faa45da74
Memory size:     1024MB
Page Fusion:     off
VRAM size:       8MB
CPU exec cap:    100%
HPET:            on
Chipset:         piix3
Firmware:        BIOS
Number of CPUs:  1
PAE:             on
Long Mode:       on
Triple Fault Reset: off
APIC:            on
X2APIC:          off
CPUID Portability Level: 0
CPUID overrides: None
Boot menu mode:  message and menu
Boot Device (1): DVD
Boot Device (2): HardDisk
Boot Device (3): HardDisk
Boot Device (4): Not Assigned
...
```

As you can see, there are many settings, but there are a few we must
change in order to proceed. They are as follows.

+ --memory: The amount of RAM allocated to the VM (in Mega Bites).
+ --boot1: The first boot device. This needs to be set to whatever
type of media your operating system install media is (usually it will
be dvd).
+ --BOOT2: The second boot device. This needs to be set to hdd, so we can boot from our
virtual hard disk after installing the operating system.

For our Debian_64 machine, we type:

```
$ VBoxManage modifyvm "Debian_64" --memory 1024 --boot1 dvd --boot2 hdd
```

Notice how we can chain arguments together. This makes it easy to
change several settings at once.

## Step 8: Starting the Machine

At this point, the machine should be completely configured---but the
operating system still needs to be installed. I'm not going to cover
how to install Debian---or any other operating system VirtualBox
supports---but you need to launch the machine before anything else.
All that is needed is the name of your virtual machine ("Debian_64" for
our example). Use this command:

```
$ VBoxManage startvm "Debian_64"
```

When installation is complete, remove the installation media from the
virtual machine with the same command used in [step 6](#step-6-adding-media-to-the-controller), except
omit the --type argument and specify none for the --medium parameter.

```
$ VBoxManage storageattach "Debian_64" --storagectl "SATA" --device 1 --port 0 --medium none
```

# Optional Steps

If you're not interested in deploying your VM to other computers, then
you're done. If you are, however, the next two steps will show you
how.

## Step 9: Exporting the Virtual Machine

We begin by exporting the virtual machine to an OVM appliance. This is done with the

```
VBoxManage export
```

 command. As always, you need to supply the name of the virtual
machine you want to export. You also need to supply the -o argument
with the path to the .ovm file that will become your exported
machine. So, for example, we can use

```
$ VBoxManage export "Debian_64" -o /Volumes/virtual_machines/exported/Debian_64.ovm
```

to export our sample Debian_64 virtual machine.

## Step 10: Importing the Virtual Machine

Once you have your .ovm file, you can import it back into VirtualBox
with the

```
VBoxManage import
```

command. All we need for this to work is the path to the .ovm file.
Thus:

```
$ VBoxManage import /Volumes/virtual_machines/exported/Debian_64.ovm
```

will import our Debian_64 virtual appliance back into VirtualBox. The
machine is already registered, so it can be started the same as in
[step 8](#step-8-starting-the-machine)

```
$ VBoxManage startvm "Debian_64"
```

# Bonus

> If you're not automating it, you're doing something wrong.

While creating at least one virtual machine by manually entering the commands is a good idea (just so you know how), no one wants to enter them over and over again. Fortunately, I have some tools to automate VM creation and management. These tools are intended for a Linux system, although some of them may run on Windows. You can find them at
<https://github.com/hadenpike/vbox_auto.git>.

## Installation

Just download the code and add the directory to your PATH environment variable. You will need [Perl] if you don't already have it.

## vbox_create

This tool automates most of the [Creating A Virtual Machine](#creating-a-virtual-machine) section. Just give it the name, OS type and the path to your installation media. The --basefolder option is set to your currrent working directory. So, to create our sample machine, we could say:

```
$ vbox_create "Debian_64" "Debian_64" /Volumes/virtual_machines/Debian_64/install_debian_64.iso
```
	    
this tool makes a few assumptions about the settings of the VM. These settings reflect my preferences. Feel free to modify the script to fit your tastes.

## vbox_clean_vmlist

If you delete, move, or otherwise relocate the virtual machine, VirtualBox doesn't know that. Consequently, if you list your virtual machines with

```
$ VBoxManage list vms
```

the system reports the machine, but it can't find the file, and thus reports it as "inaccessible". You can remove the machine with the

```
VBoxManage unregistervm
```

command and then re-register it in the new location (see [step 3](#step-3-registering-the-machine)). You must give the unregistervm command the UUID of the virtual machine. You can obtain this by looking at the output of

```
$ VBoxManage list vms
```

The vbox_clean_vmlist tool examines the output of this command and calls

```
VBoxManage unregistervm
```

on any machine reported to be inaccessible. Just run it at the command line:

```
$ vbox_clean_vmlist
```

# Conclusion

I've barely scratched the surface of what is possible with [VBoxManage]. Hopefully, this is enough to get you started though,
and the manual does not appear so daunting when you need to lookup information.
Happy VM creating.

[VirtualBox]: http://www.virtualbox.org/
[VBoxManage]: http://www.virtualbox.org/manual/ch08.html
[Perl]: http://www.perl.org/
[^1]: If you're unaware of how to convert between Gigabytes and Megabytes, multiply the number of Gigabytes by 1024.
[^2]: Turn this on unless you know why it needs to be off.
