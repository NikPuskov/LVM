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

