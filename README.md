# LVM

Стенд поднят на Vagrant (Vagrantfile прилагается) 

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
