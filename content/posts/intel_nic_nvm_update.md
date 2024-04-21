+++
title = "Intel NIC nvm update (X710-DA2, i225-T1, i210-T1)"
date = "2024-04-21T19:42:19+02:00"
author = ""
authorTwitter = ""
cover = ""
tags = ["network", "nic", "intel"]
keywords = ["", ""]
description = ""
showFullContent = false
readingTime = false
hideComments = false
+++
Modern Intel NICs have both an updatable bootloader and firmware (NVM) that can be updated. If your NIC is on a custom PCB and/or on a motherboard you may get NVM updates via BIOS and/or a special firmware package from the vendor but likely not. It's a bit unclear what the firmware updates do, the Intel changelogs are quite poor but using the 'retail' NVM firmware does seem to enable your NIC to have all the features rather than be stuck to a lockdowned subset as is common with vendor cards (OEM) commonly found on ebay from lenovo/dell/hp servers... These are obviously interesting since the real retail Intel NICs are both hard to find and even then hard to tell if they are really Intel retail cards.

All the cards I played with are from second hand sources and I've no idea if they are real, fake, etc... Note that bricking your NIC is probably not that unlikely with these steps so beware of these steps! I obviously make no guarrantees this works for you ;-)

Cards I tried this on:
- X710-DA2
- i255-T1
- i210-T1

I suspect that any modern i2xx and/or > X520, X540, X550, X710 or X810 cards should work, just note that X520 & X540 seem to be deprecated so you'll need to grab older packages.

Intel NICs seem to use [YottaMark or BradyID](https://www.intel.com/content/www/us/en/support/articles/000007074/ethernet-products/gigabit-ethernet-adapters-up-to-2-5gbe.html) which you can use to verify if your card is genuine. I've no idea how this works accurately since it seems that cloning a device would be trivial... I believe cards with this mark are typically retail units but my X710-DA2 has a yottamark sticker which can't be validated due to YottaMark being down and it seems the card is not considered retail by Intel themselves since it's not on the retail update list - so I've no idea if these are even worth looking out for. I can't imagine the fake market for Intel NICs is that high but who knows!

## How it works

Intel cards have both a bootloader (except i210) and an NVM firmware. Intel release a big all in one package called a complete driver [package](https://www.intel.com/content/www/us/en/download/15084/intel-ethernet-adapter-complete-driver-pack.html). It's unclear why they bundle it this way but here we go. Download it, there might be a newer one too. Hilariously the release notes points to a powerpoint slide that links to a PDF... utter madness. Once you have settled and decided that obviously your problem is fixed by this new version let's get to it.

## OS choice

Intel supports it seems mostly Linux or EFI shell directly for these updates and requires a special out of tree driver to communicate with the NIC. As usual the scripts are kind of brocken on Arch Linux so I decided to skip the frustration and just use a ubuntu 22.04 live USB stick to boot my desktop and use that to flash all my cards. Note that because you're building a kernel module and I didn't want to sign it, you'll need to disable secure boot.

The trick is just to boot via USB, select 'live' and then install the following packages, you really don't need much:
```{.sh}
sudo apt install gcc-12 linux-headers-$(uname -r) make ethtool
sudo update-alternatives --install /usr/bin/gcc gcc /usr/bin/gcc-12 10
```

## Updating the bootloader

Let's go check the bootloader state
```{.sh}
cd /media/ubuntu/<mydisk>/Temp/Release_29.0.1/APPS/BootUtil/Linux_x64/DRIVER/
./install
cd ../
cp ../BootImg.FLB .
sudo ./bootutil64e

Intel(R) Ethernet Flash Firmware Utility
BootUtil version 1.40.05.0
Copyright (C) 2003-2023 Intel Corporation

Type BootUtil -? for help

Port Network Address Location Series  WOL Flash Firmware                Version
==== =============== ======== ======= === ============================= =======
  1   001B21EA973B     3:00.0 Gigabit YES UEFI,CLP,PXE Enabled,iSCSI    1.5.48
  2   40A6B7ACEF53     4:00.0 Gigabit N/A FLASH Not Present
  3   047C16577E57     5:00.0 Gigabit N/A FLASH Not Present
```

Here you can see that the first card is the only one that is 'valid' here. Flash it simply with:
```{.sh}
./bootutil64e -NIC=1 -up=combo
```

Note that a dual port NIC seems to only need the first entry flashed, the 'combo' seems to be a type of firmware, I'm not entirely sure I understood the implications here. In all cases that seemed to work.

## Flashing X710-DA2

This very nice [blog post](https://gist.github.com/mietzen/736583d37a1d370273c0775aaaa57aa5) gave me the idea for flashing this card. And essentially all my next flashes where based on the ideas in this post. Essentially here my card was not retail so to flash the NVM the ethtool is really useful

```{.sh}
ubuntu@ubuntu ~ % sudo ethtool -i enp1s0f1np1
driver: i40e
version: 6.5.xxx
firmware-version: 8.15 0x8000af31 1.2877.0
expansion-rom-version: 
bus-info: 0000:01:00.1
supports-statistics: yes
supports-test: yes
supports-eeprom-access: yes
supports-register-dump: yes
supports-priv-flags: yes
```

As you can see my card is a 0x8000AF31. What you can do once you flashed a bootloader is to simply figure out what card you have in the nvmupdate.cfg and update the REPLACES entry with your card's ID. In my case it is an X710-DA2.

Flashing the generic firmware (or updating it, unclear what gives you the extra features really), means you now have SR-IOV support and full ASPM support.

## Flashing i225-T1

I think this is a retail item. Or at least I assume since the tool knew this card.

```{.sh}
cd ~/Release_29.0.1/NVMUpdatePackage/I225
tar xvf I225_NVMUpdatePackage_v1_00_Linux.tar.gz
cd I225/Linux_x64/
I225/Linux_x64$ sudo ./nvmupdate64e 

Intel(R) Ethernet NVM Update Tool
NVMUpdate version 1.41.3.1
Copyright(C) 2013 - 2024 Intel Corporation.


WARNING: To avoid damage to your device, do not stop the update or reboot or power off the system during this update.
Inventory in progress. Please wait [**........]


Num Description                          Ver.(hex)  DevId S:B    Status
=== ================================== ============ ===== ====== ==============
01) Intel(R) Ethernet Server Adapter      N/A(N/A)   1533 00:003 Update not    
    I210-T1                                                      available     
02) Intel(R) Ethernet Network Adapter   1.87(1.57)   15F2 00:004 Update        
    I225-T1                                                      available     
03) Intel(R) Ethernet Controller (3)      N/A(N/A)   15F3 00:005 Update not    
    I225-V                                                       available     

Options: Adapter Index List (comma-separated), [A]ll, e[X]it
Enter selection: 

Num Description                          Ver.(hex)  DevId S:B    Status
=== ================================== ============ ===== ====== ==============
01) Intel(R) Ethernet Server Adapter      N/A(N/A)   1533 00:003 Update not    
    I210-T1                                                      available     
02) Intel(R) Ethernet Network Adapter  1.148(1.94)   15F2 00:004 Update        
    I225-T1                                                      successful    
03) Intel(R) Ethernet Controller (3)      N/A(N/A)   15F3 00:005 Update not    
    I225-V                                                       available     
```


## Flashing i210-T1

I thought this would be a retail item but seems it is not. Also these cards seem to have no bootloader, just an NVM.

```{.sh}
ubuntu@ubuntu:/media/ubuntu/CrucialP3/Temp/Release_29.0.1/NVMUpdatePackage/I210/I210/Linux_x64$ sudo ./nvmupdate64e 

Intel(R) Ethernet NVM Update Tool
NVMUpdate version 1.40.5.5
Copyright(C) 2013 - 2023 Intel Corporation.


WARNING: To avoid damage to your device, do not stop the update or reboot or power off the system during this update.
Inventory in progress. Please wait [**|.......]


Num Description                          Ver.(hex)  DevId S:B    Status
=== ================================== ============ ===== ====== ==============
01) Intel(R) Ethernet Server Adapter    3.37(3.25)   1533 00:003 Update        
    I210-T1                                                      available     
02) Intel(R) Ethernet Network Adapter     N/A(N/A)   15F2 00:004 Update not    
    I225-T1                                                      available     
03) Intel(R) Ethernet Controller (3)      N/A(N/A)   15F3 00:005 Update not    
    I225-V                                                       available     

Options: Adapter Index List (comma-separated), [A]ll, e[X]it
Enter selection: 1
Would you like to back up the NVM images? [Y]es/[N]o: Y
Update in progress. This operation may take several minutes.
[....******]


Num Description                          Ver.(hex)  DevId S:B    Status
=== ================================== ============ ===== ====== ==============
01) Intel(R) Ethernet Server Adapter    3.48(3.30)   1533 00:003 Update        
    I210-T1                                                      successful    
02) Intel(R) Ethernet Network Adapter     N/A(N/A)   15F2 00:004 Update not    
    I225-T1                                                      available     
03) Intel(R) Ethernet Controller (3)      N/A(N/A)   15F3 00:005 Update not    
    I225-V                                                       available     

A power cycle is required to complete the update process.

Tool execution completed with the following status: All operations completed successfully.
Press any key to exit.
```
