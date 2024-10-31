# Unofficial Patch for pfsense 2.7.2 to add support for ED25519 type certificates

_A patch to add ED25519 certificate support in pfSense, so you can import strange OpenVPN profiles._


## Overview  
This patch addresses the lack of ED25519 certificate support in **pfSense**. It enables importing OpenVPN profiles that use ED25519 certificates, such as those from **HackTheBox**. 
This is a hack job patch, but I have tried to maintain the core checking logic that pfsense uses regarding checking for weak certs. 
FYI I am not a developer, but I was able to make this work.

THIS SHOULD NOT BE DEPLOYED IN PRODUCTION, IF YOU DO, IT'S YOUR FAULT

When attempting to create the VPN profile, the "Client Certificate" option does not list the HTB certs you previously imported. This is because pfsense checks for valid certs and certs without weak algorithms.

The change involves modifying a single file:  
`/etc/inc/certs.inc`

For reference, the issue is discussed in the following resources:  
- [Netgate Forum: OpenVPN ED Cert](https://forum.netgate.com/topic/182698/openvpn-ed-cert)  
- [Reddit Discussion](https://www.reddit.com/r/PFSENSE/comments/1gdjqru/comment/lu8igt4/)  
- [Ben Heater: pfSense + HackTheBox OpenVPN](https://benheater.com/pfsense-hackthebox-openvpn-nat/)  
- [pfSense Redmine Issue #14762](https://redmine.pfsense.org/issues/14762)

Special Thanks to Ben Heater for his blog on how to setup HTB profiles on pfsense and his initial work figuring out why it doesn't work on newer versions.

## Changes Made
You can obviously diff this but here is the short version:

There is a function called **cert_build_list**
pfsense uses this to build the list of certs it uses.
It runs a compatibility check via **cert_check_pkey_compatibility**
This check compares the cert consumer and features against the array **$cert_curve_compatible**

I've added a new function **cert_get_ed25519** to check the public key for ED25519 support
Then implemented the new check into the existing logic.
I have also updated the **$cert_curve_compatible** array and included ED25519 under the OpenVPN consumer.

### Line Refs:
 - 2678 - Changes made to cert_build_list
 - 2692 - Start of new function **cert_get_ed25519**
 - 2575 - Updates to the **$cert_curve_compatible** array

## Installation  
1. Download the patched version of `certs.inc` from this repository.
2. Obviously make a backup of your original file
3. Replace the existing file on your pfSense system:  
   ```bash
   cp certs.inc /etc/inc/certs.inc
4. Use the cli shell option 16 to restart php-fpm
5. You didn't make a backup did you?
6. I AM NOT A DEVELOPER, I JUST MADE THIS WORK FOR ME

## Usage
After installing the patch, reboot services and reimport your profile as per Ben Heater's blog and you shouldn't have an issue

## Compatibility
- **pfSense Version:** Tested with version 2.7.2 

## License  
Don't care, just thank me somewhere if this helps you.

## Acknowledgements  
- **Netgate Forums** for discussion and insights on the issue.  
- **Ben Heater** for documentation on pfSense and HackTheBox setup and his initial investigations. 
