# cosm-public ipxe readme

## Usage

### Set up iPXE boot menu
```bash
cd ~/ipxe/src
nano embed.ipxe
```

### embed.ipxe for PXE boot:
```ipxe
#!ipxe
dhcp && goto netboot || goto dhcperror

:dhcperror
prompt --key s --timeout 10000 DHCP failed, hit 's' for the iPXE shell; reboot in 10 seconds && shell || reboot

:netboot
chain tftp://${next-server}/main.ipxe ||
prompt --key s --timeout 10000 Chainloading failed, hit 's' for the iPXE shell; reboot in 10 seconds && shell || reboot
```

### embed.ipxe for HTTP boot:
```ipxe
#!ipxe
dhcp && goto netboot || goto dhcperror

:dhcperror
prompt --key s --timeout 10000 DHCP failed, hit 's' for the iPXE shell; reboot in 10 seconds && shell || reboot

:netboot
chain http://192.168.1.1/main.ipxe ||
prompt --key s --timeout 10000 Chainloading failed, hit 's' for the iPXE shell; reboot in 10 seconds && shell || reboot
```

## Add ping and NTFS support to iPXE
```bash
cd ~/ipxe/src

# NTFS support
sed -i 's/#undef\tDOWNLOAD_PROTO_NFS/#define\tDOWNLOAD_PROTO_NFS/' config/general.h

# Ping support
sed -i 's/\/\/#define\ PING_CMD/#define\ PING_CMD/' config/general.h
sed -i 's/\/\/#define\ IPSTAT_CMD/#define\ IPSTAT_CMD/' config/general.h
sed -i 's/\/\/#define\ REBOOT_CMD/#define\ REBOOT_CMD/' config/general.h
sed -i 's/\/\/#define\ POWEROFF/#define\ POWEROFF/' config/general.h
```

## Build iPXE
```bash
cd ~/ipxe/src
make bin-x86_64-efi/ipxe.efi EMBED=embed.ipxe
```

## Copy iPXE network boot firmware

> **Note:** This should be served via TFTP or HTTP (TFTP for iPXE boot, NGINX for HTTP boot)
```bash
cp ~/ipxe/src/bin-x86_64-efi/ipxe.efi /pxe-boot/ipxe.efi
```

## Hello World iPXE Menu
```bash
nano /pxe-boot/main.ipxe
```
```ipxe
#!ipxe
:MENU
menu
item --gap -- ---------------- iPXE boot menu ----------------
item hello        Hello world
item shell        ipxe shell
choose --default return --timeout 5000 target && goto ${target}

:hello
echo "hello world"
boot ||
goto MENU

:shell
shell ||
goto MENU

autoboot
```

## Additional Resources

- [wimboot](https://github.com/ipxe/wimboot) - Wimboot is fast way to boot into Windows PE using HTTP
- [Creating WinPE bootable drive](https://learn.microsoft.com/en-us/windows-hardware/manufacture/desktop/winpe-create-usb-bootable-drive?view=windows-11) - Creating WINPE wim file
- [Dell Drivers](https://www.dell.com/support/home/en-us?app=drivers) - Drivers for Dell needed for WinPE to access drives and networking
- [Adding drivers to WIM files](https://learn.microsoft.com/en-us/windows-hardware/manufacture/desktop/add-and-remove-drivers-to-an-offline-windows-image?view=windows-11) - Adding drivers to WIM files
- [iPXE Documentation](https://ipxe.org/) - Reference for iPXE related documentation, error codes, and writing iPXE scripts

=======
iPXE network bootloader
=======================

[![Build](https://img.shields.io/github/actions/workflow/status/ipxe/ipxe/build.yml)](https://github.com/ipxe/ipxe/actions/workflows/build.yml?query=branch%3Amaster)
[![Coverity](https://img.shields.io/coverity/scan/12130)](https://scan.coverity.com/projects/ipxe-ipxe)
[![Release](https://img.shields.io/github/v/release/ipxe/ipxe)](https://github.com/ipxe/ipxe/releases/latest)

iPXE is the leading open source network boot firmware. It provides a
full PXE implementation enhanced with additional features such as:

 - boot from a web server via HTTP or [HTTPS][crypto],

 - boot from an iSCSI, FCoE, or AoE [SAN][sanboot],

 - control the boot process with a [script][scripting],

 - create interactive [forms][forms] and [menus][menus].

You can use iPXE to replace the existing PXE ROM on your network card,
or you can chainload into iPXE to obtain the features of iPXE without
the hassle of reflashing.

iPXE is free, open-source software licensed under the GNU GPL (with
some portions under GPL-compatible licences).

You can download the [rolling release binaries][rolling] (built from
the latest commit), or use the most recent [stable release][release].

For full documentation, visit the [iPXE website][ipxe].


[crypto]: https://ipxe.org/crypto
[forms]: https://ipxe.org/cmd/present
[ipxe]: https://ipxe.org
[menus]: https://ipxe.org/cmd/choose
[release]: https://github.com/ipxe/ipxe/releases/latest
[rolling]: https://boot.ipxe.org
[sanboot]: https://ipxe.org/cmd/sanboot
[scripting]: https://ipxe.org/scripting

