# HP-Spectre-x360-Convertible-i7-6500u-Opencore-Hackintosh-EFI
Use this only for this laptop. It probably won't work on other laptops.

**CAUTION: YOU MUST HAVE A BASIC UNDERSTANDING OF EFI's, YOUR COMPUTER's ESP, THE COMMAND-LINE, ETC.**
Use this [guide](https://dortania.github.io/OpenCore-Install-Guide/) for troubleshooting.

## Requirements
- 4 GB or higher USB stick (Any data on this USB will be lost)
- If installing Sequoia or Tahoe, you will need ethernet or an ethernet dongle (or Horndis) because there is no Airportitlwm.kext for those versions, and itlwm.kext is unsupported in macOS Recovery.
- Compatible wifi card or an ethernet port, or you can tether with an android device using Horndis.
- Python installed.
- Working OS (Windows or Linux)

This repo contains the EFI folder and macrecovery.py

## Steps
### 1. Creating/Formatting the USB
  - Windows
      1. Open cmd.exe (command prompt) and navigate to the folder that you downloaded from here.
      2. Now you will need to run one of these commands to download the vesion you want: 
      
        # Mavericks (10.9):
        py macrecovery.py -b Mac-F60DEB81FF30ACF6 -m 00000000000FNN100 download
        
        # Yosemite (10.10):
        py macrecovery.py -b Mac-E43C1C25D4880AD6 -m 00000000000GDVW00 download
        
        # El Capitan (10.11):
        py macrecovery.py -b Mac-FFE5EF870D7BA81A -m 00000000000GQRX00 download
        
        # Sierra (10.12):
        py macrecovery.py -b Mac-77F17D7DA9285301 -m 00000000000J0DX00 download
        
        # High Sierra (10.13)
        py macrecovery.py -b Mac-7BA5B2D9E42DDD94 -m 00000000000J80300 download
        py macrecovery.py -b Mac-BE088AF8C5EB4FA2 -m 00000000000J80300 download
        
        # Mojave (10.14)
        py macrecovery.py -b Mac-7BA5B2DFE22DDD8C -m 00000000000KXPG00 download
        
        # Catalina (10.15)
        py macrecovery.py -b Mac-00BE6ED71E35EB86 -m 00000000000000000 download
        
        # Big Sur (11)
        py macrecovery.py -b Mac-42FD25EABCABB274 -m 00000000000000000 download
        
        # Monterey (12)
        py macrecovery.py -b Mac-FFE5EF870D7BA81A -m 00000000000000000 download
        
        # Ventura (13)
        py macrecovery.py -b Mac-4B682C642B45593E -m 00000000000000000 download
        
        # Sonoma (14)
        py macrecovery.py -b Mac-226CB3C6A851A671 -m 00000000000000000 download
        
        # Latest version
        # ie. Sequoia (15)
        py macrecovery.py -b Mac-937A206F2EE63C01 -m 00000000000000000 download
    
      3. Now you will get either `BaseSystem.dmg` and `BaseSystem.chunklist` or `RecoveryImage.dmg` and `RecoveryImage.chunklist`.
      4. Download [Rufus](https://rufus.ie/en/). Select your USB drive and set the Boot selection as non bootable. Set the filesystem as Large FAT32, then once it is done delete the "autorun" files in the USB Drive.
      5. Now go to the root of the USB drive and create a folder called `com.apple.recovery.boot` and drag your .dmg and .chunklist files in there.
      6. Now grab the EFI folder and place it at the root of the USB drive alongside that folder we just created.
  - Linux
      1. `cd` into the folder that you downloaded from here.
      2. Look at the "Windows section", particularly the commands to enter into `cmd.exe`. Enter that exact command into the terminal, but replace the `py` at the beginning with `python3` and add a `./` before `macrecovery.py` to execute it.
      3. Now you will get either a `BaseSystem.dmg` & `BaseSystem.chunklist` or a `RecoveryImage.dmg` and `RecoveryImage.chunklist`.
      4. Now if your are experienced, follow steps 5-16, but if you aren't follow steps 17-X.
      5. Run `lsblk` in the terminal. Determine your USB Device block. If you don't know what this command does, I highly suggest you follow steps 17-X.
      6. Run `sudo gdisk /dev/<your USB block>`
      7. If it asks you what partition table to use, select `GPT` (it never asked me when I did it 😞)
      8. Send `o` to clear the partition table and make a new GPT one. Confirm with `y`
      9. Send `n` for a new partition:
          - Keep the partition number and first sector blank for default.
          - Last sector: `+200M` (will create a 200 MB partition where we'll put our EFI folder)
          - Hex code or GUID: `0700`
      10. Send `n` again for a new partition
          - Keep the partition number and first sector blank for default.
          - Last sector keep blank for default
          - Hex cod or GUID: `af00`.
      11. Send `w` and confirm with `y`.
      12. Close `gdisk` by sending `q`.
      13. Run `lsblk` again to determine the 200MB partition and the other one.
      14. Run `sudo mkfs.vfat -F 32 -n "OPENCORE" /dev/<your 200MB partition>` This will name the 200 MB block "OPENCORE" and format it as FAT32.
      15. Now get those `.dmg` and `.chunklist` files that we created earlier and place them in a folder called `com.apple.recovery.boot` the root of the FAT32 partition we created earlier.
      16. Now place the EFI folder on the root of the "OPENCORE" partition (nothing else should be there).
      17. Now for you inexperienced folks...
      18. Open your distro's partition manager and look for the USB drive (it will show up on the left depending on your desktop environment).
      19. Click "New Partition Table" or something like that and click "GPT".
      20. Now create a new partition that is FAT32 and 200 MB in size, name it "OPENCORE".
      21. Now, figure out what your USB block is. It is most likely `/dev/sdb` but please double check.
      22. Open terminal and type `sudo gdisk /dev/<your USB block> `. Type `n` for a new partition. Leave "Partition number", "First sector" and "Last sector" blank. Set the "Hex code or GUID" as `af00`. Send `w` and confirm with `y`.
      23. Now `cd` to the root of the larger partition of your USB. Run `mkdir com.apple.recovery.boot`.
      24. Copy the `.dmg` and `.chunklist` files into it.
      25. Copy the EFI folder into the 200 MB partiion named as "OPENCORE".
### 2. USB Port Mapping (only supports Windows)
  1. Our EFI folder already has `USBToolBox.kext`, which is required. Now we need to make `UTBMap.kext`. Follow this [guide](https://github.com/USBToolBox/tool?tab=readme-ov-file#usage) on how to make it until step 7 (download `Windows.exe`).
  2. Once you have the `UTBMap.kext` go into the "OPENCORE" partition of the USB drive and into the EFI -> OC -> Kexts, and copy it into there.
  3. If you are one of my friends on Linux, you will have to map after installing. Grab the [USBToolBox Release](https://github.com/USBToolBox/kext/releases/download/1.2.0/USBToolBox-1.2.0-RELEASE.zip) and grab UTBDefault.kext from there and place it into the Kexts folder. You will also need to update config.plist.
