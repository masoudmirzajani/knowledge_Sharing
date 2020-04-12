Change root Password "linux"
========================


on boot menu press e

end of UTF-8 

type  rd.break

mount -o remount,rw /sysroot

chroot /sysroot

passwd root

new pass:

retype new pass:


 touch /.autorelabel
 
 exit
 
 exit
 