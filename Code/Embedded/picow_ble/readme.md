with raspberry pi pico vs code extension installed select compile on the bottom ribbon

mount the mass storage device manually:
$ dmesg | tail
[ 371.973555] sd 0:0:0:0: [sda] Attached SCSI removable disk
$ sudo mkdir -p /mnt/pico
$ sudo mount /dev/sda1 /mnt/pico
If you can see files in /mnt/pico, the USB Mass Storage Device has mounted correctly:
$ ls /mnt/pico/
INDEX.HTM INFO_UF2.TXT
Copy your blink.uf2 onto the device:
$ sudo cp blink.uf2 /mnt/pico
$ sudo sync
The microcontroller automatically disconnects as a USB Mass Storage Device and runs your code, but just to be safe,
you should unmount manually as well:
$ sudo umount /mnt/pico