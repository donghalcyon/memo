## mount usb
sudo fdisk -l
mkdir /home/pi/usb
sudo mount /dev/sda1 /home/pi/usb
## umount usb
sudo umount /home/pi/usb
