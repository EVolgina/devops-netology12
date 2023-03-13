6 mdadm --create --verbose /dev/md1 -l 1 -n 2 /dev/sd{b1,c1}
7 mdadm --create --verbose /dev/md2 -l 0 -n 2 /dev/sd{b2,c2}:
