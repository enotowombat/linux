### Homework 3


#### Задание

Работа с LVM
на имеющемся образе
/dev/mapper/VolGroup00-LogVol00 38G 738M 37G 2% /

уменьшить том под / до 8G
выделить том под /home
выделить том под /var
/var - сделать в mirror
/home - сделать том для снэпшотов
прописать монтирование в fstab
попробовать с разными опциями и разными файловыми системами ( на выбор)
- сгенерить файлы в /home/
- снять снэпшот
- удалить часть файлов
- восстановится со снэпшота
- залоггировать работу можно с помощью утилиты screen

* на нашей куче дисков попробовать поставить btrfs/zfs - с кешем, снэпшотами - разметить здесь каталог /opt


#### Основная часть

- Создаем временные lv для хранения xfs дампа и / раздела
- Делаем дамп, восстанавливаем уже в уменьшенном размере в новый /
- Делаем chroot в новый / раздел, генерим оттуда grub.cfg(и меняем там /) и initramfs 

```
sudo su
pvcreate /dev/sde
vgcreate vg0 /dev/sde
lvcreate -L 4G -n tmpLV vg0
mkfs.xfs /dev/vg0/tmpLV
mount /dev/vg0/tmpLV /tmp
yum install xfsdump -y
xfsdump -l 0 -L "root backup" -M "root backup" -f /tmp/root.dump /
lvcreate -L 4G -n tmpLVroot vg0
mkfs.xfs /dev/vg0/tmpLVroot
mount /dev/vg0/tmpLVroot /mnt
xfsrestore -f /tmp/root.dump /mnt
mount --bind /proc /mnt/proc
mount --bind /dev /mnt/dev
mount --bind /sys /mnt/sys
mount --bind /run /mnt/run
mount --bind /boot /mnt/boot
chroot /mnt/
grub2-mkconfig -o /boot/grub2/grub.cfg
dracut /boot/initramfs-3.10.0-693.21.1.el7.x86_64.img $(uname -r) --force
sed -i 's/rd.lvm.lv=VolGroup00\/LogVol00/rd.lvm.lv=vg0\/tmpLVroot/' /boot/grub2/grub.cfg
exit
reboot
```

- Загружаемся с новым /, уменьшаем размер lv старого /, создаем новые /var /home, старые удаляем, переключаемся обратно на старый / так же, как в прошлый раз

```
sudo su
lvremove /dev/VolGroup00/LogVol00 -y
lvcreate -n VolGroup00/LogVol00 -L 8G /dev/VolGroup00 -y
mkfs.xfs /dev/VolGroup00/LogVol00
mount /dev/VolGroup00/LogVol00 /mnt
mount /dev/vg0/tmpLV /tmp
xfsrestore -f /tmp/root.dump /mnt
mount --bind /proc /mnt/proc
mount --bind /dev /mnt/dev
mount --bind /sys /mnt/sys
mount --bind /run /mnt/run
mount --bind /boot /mnt/boot
vgextend vg0 /dev/sdb
lvcreate -l 100%FREE -m1 -n var vg0
mkfs.xfs /dev/vg0/var
mkdir /new_var
mount /dev/vg0/var /new_var
cp -ar /var/* /new_var
lvcreate -L 8G -n home VolGroup00
mkfs.xfs /dev/VolGroup00/home
mkdir /new_home
mount /dev/VolGroup00/home /new_home
cp -ar /home/* /new_home
chroot /mnt/
grub2-mkconfig -o /boot/grub2/grub.cfg
dracut /boot/initramfs-3.10.0-693.21.1.el7.x86_64.img $(uname -r) --force
echo "/dev/mapper/VolGroup00-home /home xfs defaults 0 0" >> /etc/fstab
echo "/dev/mapper/vg0-var /var xfs defaults 0 0" >> /etc/fstab
exit
rm -rf /mnt/var
rm -rf /mnt/home
rm -f /tmp/root.dump
reboot
```
Проверяем, что удалось загрузиться и все нормально. Ненужные lv удаляем
```
sudo su
lvremove /dev/vg0/tmpLV -y
lvremove /dev/vg0/tmpLVroot -y

```
####Снапшоты
- сгенерить файлы в /home/, снять снэпшот, удалить часть файлов, восстановится со снэпшота

```
touch /home/file{1..10}
ls /home
lvcreate -L 8GB -s -n home-snap0 /dev/VolGroup00/home
rm -rf /home/file{1..5}
ls /home
lvconvert --merge VolGroup00/home-snap0
```
/home используется, без перезагрузки восстановиться не удалось
```
  Can't merge until origin volume is closed.
  Merging of snapshot VolGroup00/home-snap0 will occur on next activation of VolGroup00/home.
```
`reboot`
После перезагрузки файлы восстановлены


### ZFS

[Документация](https://github.com/zfsonlinux/zfs/wiki/Admin-Documentation)

#### Install

Заработало не сразу, на всякий случай короткое описание проблемы, вдруг пригодится:
`modprobe: FATAL: Module zfs not found`
Модули добавились, но не собрались
`# dkms status`
`spl, 0.7.9: added`
`zfs, 0.7.9: added`
На `dkms install`, ошибка:
```
Your kernel headers for kernel 3.10.0-693.21.1.el7.x86_64 cannot be found at /lib/modules/3.10.0-693.21.1.el7.x86_64/build or /lib/modules/3.10.0-693.21.1.el7.x86_64/source
```
Ядро на самом деле уже было новее (откуда взялось обновление ядра без обновления grub конфига в вагрант боксе не знаю), но ОС грузилась со старого. Суть расхождений версий ядра, почему не собиралось на старой версии, не совсем понял. 
Обновил конфиг grub, загрузился на новой версии, сделал `dkms install`

```
# dkms status
spl, 0.7.9, 3.10.0-862.2.3.el7.x86_64, x86_64: installed
zfs, 0.7.9, 3.10.0-862.2.3.el7.x86_64, x86_64: installed
```
`modprobe zfs`

Далее все ОК


#### Create pool

Создаем пул из трех дисков: два в зеркало, третьий кэш, монтируем в /opt

```
# zpool create pool mirror sdb sdc cache sdd -m /opt
# zpool status pool
  pool: pool
 state: ONLINE
  scan: none requested
config:

        NAME        STATE     READ WRITE CKSUM
        pool        ONLINE       0     0     0
          mirror-0  ONLINE       0     0     0
            sdb     ONLINE       0     0     0
            sdc     ONLINE       0     0     0
        cache
          sdd       ONLINE       0     0     0

errors: No known data errors
# zfs list
NAME   USED  AVAIL  REFER  MOUNTPOINT
pool    96K   112M    24K  /opt
```

#### Snapshots

Добавляем файлы, создаем снепшот, удалям часть файлов, восстанавливаем

```
# touch /opt/file{1..10}
# zfs snapshot pool@test_snap
# zfs list -t snapshot
NAME             USED  AVAIL  REFER  MOUNTPOINT
pool@test_snap     0B      -    32K  -
# rm -rf /opt/file{1..5}
# zfs list -t snapshot
NAME             USED  AVAIL  REFER  MOUNTPOINT
pool@test_snap  19.5K      -    32K  -
# zfs rollback pool@test_snap
# ls /opt
file1  file10  file2  file3  file4  file5  file6  file7  file8  file9
# zfs list -t snapshot
NAME             USED  AVAIL  REFER  MOUNTPOINT
pool@test_snap    13K      -    32K  -
```
