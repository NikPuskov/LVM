# LVM

Стенд поднят на Vagrant (Vagrantfile прилагается)

![Image alt](https://github.com/NikPuskov/LVM/blob/main/lvm.jpg)

I) Уменьшить том под / до 8G

1. Подготовим временный том для / раздела

`pvcreate /dev/sdb`

`vgcreate vg_root /dev/sdb`

`lvcreate -n lv_root -l +100%FREE /dev/vg_root`

2. Создадим на нем файловую систему и смонтируем его, чтобы перенести туда данные

`mkfs.ext4 /dev/vg_root/lv_root`

`mount /dev/vg_root/lv_root /mnt`

3. Копируем все данные с / раздела в /mnt

`rsync -avxHAX --progress / /mnt/`

4. Проверим что скопировалось

`ls /mnt`

![Image alt](https://github.com/NikPuskov/LVM/blob/main/lvm1.jpg)

5. Сконфигурируем grub для того, чтобы при старте перейти в новый /. Сымитируем текущий root, сделаем в него chroot и обновим grub

`for i in /proc/ /sys/ /dev/ /run/ /boot/; \
 do mount --bind $i /mnt/$i; done`

`chroot /mnt/`

`grub-mkconfig -o /boot/grub/grub.cfg`

6. Обновим образ initrd

`update-initramfs -u`

7. Выйдем из под chroot и перезагрузимся

`exit`

`reboot`

![Image alt](https://github.com/NikPuskov/LVM/blob/main/lvm2.jpg)

8. После перезагрузки

![Image alt](https://github.com/NikPuskov/LVM/blob/main/lvm3.jpg)

9. Удаляем старый LV размером в 31G и создаём новый на 8G

`lvremove /dev/ubuntu-vg/ubuntu-lv`

`lvcreate -n ubuntu-vg/ubuntu-lv -L 8G /dev/ubuntu-vg`

10. Делаем на новом разделе те же операции, что и в первый раз

`mkfs.ext4 /dev/ubuntu-vg/ubuntu-lv`

`mount /dev/ubuntu-vg/ubuntu-lv /mnt`

`rsync -avxHAX --progress / /mnt/`

![Image alt](https://github.com/NikPuskov/LVM/blob/main/lvm4.jpg)

11. Ещё раз cконфигурируем grub

`for i in /proc/ /sys/ /dev/ /run/ /boot/; \
 do mount --bind $i /mnt/$i; done`

 `chroot /mnt/`

 `grub-mkconfig -o /boot/grub/grub.cfg`

 `update-initramfs -u`

 ![Image alt](https://github.com/NikPuskov/LVM/blob/main/lvm5.jpg)

 Не перезагружаемся и не выходим из под chroot - перенесем /var

 II) Выделить том под /var в зеркало

 1. На свободных дисках создаем зеркало

`pvcreate /dev/sdc /dev/sdd`

`vgcreate vg_var /dev/sdc /dev/sdd`

`lvcreate -L 950M -m1 -n lv_var vg_var`

2. Создаем на нем ФС и перемещаем туда /var

`mkfs.ext4 /dev/vg_var/lv_var`

`mount /dev/vg_var/lv_var /mnt`

`cp -aR /var/* /mnt/`

3. Сохраняем содержимое старого var

`mkdir /tmp/oldvar && mv /var/* /tmp/oldvar`

4. Монтируем новый var в каталог /var

`umount /mnt`

`mount /dev/vg_var/lv_var /var`

5. Правим fstab для автоматического монтирования /var

`echo "`blkid | grep var: | awk '{print $2}'` \
 /var ext4 defaults 0 0" >> /etc/fstab`

 6. Выходим и перезагружаемся

`exit`

`reboot`

![Image alt](https://github.com/NikPuskov/LVM/blob/main/lvm6.jpg)

7. Удаляем временную Volume Group

`lvremove /dev/vg_root/lv_root`

`vgremove /dev/vg_root`

`pvremove /dev/sdb`

8. Проверяем

`lsblk`

![Image alt](https://github.com/NikPuskov/LVM/blob/main/lvm7.jpg)

III) Выделить том под /home

1. Выделяем том под /home по тому же принципу что делали для /var

`lvcreate -n LogVol_Home -L 2G /dev/ubuntu-vg`

`mkfs.ext4 /dev/ubuntu-vg/LogVol_Home`

`mount /dev/ubuntu-vg/LogVol_Home /mnt/`

`cp -aR /home/* /mnt/`

`rm -rf /home/*`

`umount /mnt`

`mount /dev/ubuntu-vg/LogVol_Home /home/`

2. Правим fstab для автоматического монтирования /home

`echo "`blkid | grep Home | awk '{print $2}'` \
 /home xfs defaults 0 0" >> /etc/fstab`

![Image alt](https://github.com/NikPuskov/LVM/blob/main/lvm8.jpg)

