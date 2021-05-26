# stands-03-lvm

Стенд для домашнего занятия "Файловые системы и LVM"

1. уменьшить том под / до 8G

Из-за файловой системы xfs использование команды lvreduce убивало виртуалку, выдерживало уменьшение до 20G без проблем, дальше смерть, пришлось переносить все данные на другой диск, уменьшать этот, а потом возвращать обратно

`sudo yum install xfsdump`

`sudo pvcreate /dev/sdb`

`sudo vgcreate vg_tmp_root /dev/sdb`

`sudo mkfs.xfs /dev/vg_tmp_root/lv_tmp_root`

`sudo mount /dev/vg_tmp_root/lv_tmp_root /mnt`

`sudo xfsdump -J - /dev/VolGroup00/LogVol00 | sudo xfsrestore -J - /mnt`

`for i in /proc/ /sys/ /dev/ /run/ /boot/; do sudo mount --bind $i /mnt/$i; done`

`sudo chroot /mnt/`

`sudo grub2-mkconfig -o /boot/grub2/grub.cfg`

`cd /boot ; for i in `ls initramfs-*img`; do dracut -v $i `echo $i|sed "s/initramfs-//g; s/.img//g"` --force; done`

`sudo vi /boot/grub2/grub.cfg`

`sudo reboot`

`sudo lvreduce -L 8G /dev/mapper/VolGroup00-LogVol00`

`sudo mkfs.xfs /dev/VolGroup00/LogVol00 -f`

`sudo mount /dev/VolGroup00/LogVol00 /mnt`

`sudo xfsdump -J - /dev/vg_tmp_root/lv_tmp_root | sudo xfsrestore -J - /mnt`

`for i in /proc/ /sys/ /dev/ /run/ /boot/; do sudo mount --bind $i /mnt/$i; done`

`sudo chroot /mnt/`

`sudo grub2-mkconfig -o /boot/grub2/grub.cfg`

`cd /boot ; for i in `ls initramfs-*img`; do dracut -v $i `echo $i|sed "s/initramfs-//g; s/.img//g"` --force; done`

`sudo vi /boot/grub2/grub.cfg`

`sudo reboot`

`sudo lvremove /dev/vg_tmp_root/lv_tmp_root`

2. выделить том под /home

`lvcreate -n LogVol_Home -L 2G /dev/VolGroup00`

`mkfs.xfs /dev/VolGroup00/LogVol_Home`

`mount /dev/VolGroup00/LogVol_Home /mnt/`

`cp -aR /home/* /mnt/`

`rm -rf /home/*`

`umount /mnt`

`mount /dev/VolGroup00/LogVol_Home /home/`

`echo "`blkid | grep Home | awk '{print $2}'` /home xfs defaults 0 0" >> /etc/fstab`

3. выделить том под /var (/var -  сделать в mirror)

`pvcreate /dev/sdc /dev/sdd`

`vgcreate vg_var /dev/sdc /dev/sdd`

`lvcreate -L 950M -m1 -n lv_var vg_var`

`mkfs.ext4 /dev/vg_var/lv_var`

`mount /dev/vg_var/lv_var /mnt`

`cp -aR /var/* /mnt/ # rsync -avHPSAX /var/ /mnt/`

`umount /mnt`

`mount /dev/vg_var/lv_var /var`

`echo "`blkid | grep var: | awk '{print $2}'` /var ext4 defaults 0 0" >> /etc/fstab`

4. для /home - сделать том для снэпшотов

`lvcreate -n LogVol_SnapHome -L 2G /dev/VolGroup00`

5. прописать монтирование в fstab (попробовать с разными опциями и разными файловыми системами на выбор)

`echo "`blkid | grep Home | awk '{print $2}'` /home xfs defaults 0 0" >> /etc/fstab`

`echo "`blkid | grep var: | awk '{print $2}'` /var ext4 defaults 0 0" >> /etc/fstab`

6. Работа со снапшотами:
  - сгенерировать файлы в /home/

`cd /`

`sudo touch /home/file{1..20}`

  - снять снэпшот

`sudo lvcreate -L 100MB -s -n home_snap /dev/VolGroup00/LogVol_Home`

  - удалить часть файлов

`sudo rm -f /home/file*`

  - восстановиться со снэпшота

`sudo umount /home`

`sudo lvconvert --merge /dev/VolGroup00/home_snap`

`sudo mount /home`

![proof](https://user-images.githubusercontent.com/10125092/119709380-d0dc3980-be65-11eb-964e-61cbf12cb297.PNG)
