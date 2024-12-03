sudo depmod -a
sudo mkinitcpio -P

#### manually reboot when grub is being retarded
bind the hd0 and gpt with ls
set root=(hd0,gpt3) whatever it is
linux /boot/image root=/dev/sda3 (whaterver it is) 'ro'
initrd /boot/initramfs
boot