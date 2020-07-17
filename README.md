## Windows boot partition repair process

#### Format the boot partition
```sh
diskpart

list disk
# Select the disk containing the boot partiton and windows installation
select disk X

list part
# Select the boot partition
select part Y

# Delete it
delete part override

# Create and format a new partition
create part EFI
list part
# Make sure it's selected
format quick fs=FAT32 label="NeoBoot"

# Make sure the partition volume is selected
list vol
# Assign a letter to it
assign letter=W:
```

#### Copy EFI files from Windows install (C:) to boot partition (W:)
```sh
# Create the following folder structure on the fresh partition
mkdir W:\EFI\Microsoft\Boot
# Assuming C:\ is where the windows install is
# Populate the partition
xcopy /s C:\Windows\Boot\EFI\*.* W:\EFI\Microsoft\Boot
```

#### Configure the boot loader (W:)
```sh
# CD to boot partition
W:
cd EFI\Microsoft\Boot

# Initialize the store
bcdedit /createstore BCD

# Create a bootloader entry
bcdedit /store BCD /create {bootmgr} /d "neoBoot Manager"
# Create an os entry
bcdedit /store BCD /create /d "Windows 10" /application osloader

# <GUID> = value returned by the previous command
# Set bootloader values
# Default os entry
bcdedit /store BCD /set {bootmgr} default <GUID>
# Bootloader file
bcdedit /store BCD /set {bootmgr} path \EFI\Microsoft\Boot\bootmgfw.efi
# Make the new entry boot first
bcdedit /store BCD /set {bootmgr} displayorder {default}

# Set os entry values
# Windows partition
bcdedit /store BCD /set {default} device partition=C:
bcdedit /store BCD /set {default} osdevice partition=C:
# Osloader file
bcdedit /store BCD /set {default} path \Windows\System32\winload.efi
# Root path
bcdedit /store BCD /set {default} systemroot \Windows
exit
# Reboot to BIOS
# Set the new entry as default
# Boot
```
