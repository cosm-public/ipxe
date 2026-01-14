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
