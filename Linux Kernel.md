Linux Kernel
========================


**change kernel on default boot**

/etc/grub2.cfg -> /boot/grub2/grub.cfg

/etc/grub2/grubenv

awk -F\ /^menuentry/{print\<42} /etc/grub2.cfg

uname -a
grub2-set-default 0 or 1 or 2 ...
grep saved /boot/grub2/grubenv
grub2-mkconfig -o /boot/grub2/grub.cfg
reboot



