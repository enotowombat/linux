# Homework 2

```
работа с mdadm. 
добавить в Vagrantfile еще дисков
сломать/починить raid
собрать R0/R5/R10 на выбор
прописать собранный рейд в конф, чтобы рейд собирался при загрузке
создать GPT раздел и 5 партиций

в качестве проверки принимаются - измененный Vagrantfile, скрипт для создания рейда, конф для автосборки рейда при загрузке
* доп. задание - Vagrantfile, который сразу собирает систему с подключенным рейдом
```



### добавить в Vagrantfile еще дисков

В Vagrantfile добавляем 10 дисков
`vagrant up`

```
# lsblk
NAME                    MAJ:MIN RM  SIZE RO TYPE MOUNTPOINT
sda                       8:0    0   40G  0 disk 
|-sda1                    8:1    0    1M  0 part 
|-sda2                    8:2    0    1G  0 part /boot
`-sda3                    8:3    0   39G  0 part 
  |-VolGroup00-LogVol00 253:0    0 37.5G  0 lvm  /
  `-VolGroup00-LogVol01 253:1    0  1.5G  0 lvm  [SWAP]
sdb                       8:16   0  250M  0 disk 
sdc                       8:32   0  250M  0 disk 
sdd                       8:48   0  250M  0 disk 
sde                       8:64   0  250M  0 disk 
sdf                       8:80   0  250M  0 disk 
sdg                       8:96   0  250M  0 disk 
sdh                       8:112  0  250M  0 disk 
sdi                       8:128  0  250M  0 disk 
sdj                       8:144  0  250M  0 disk 
sdk                       8:160  0  250M  0 disk 
```


### собрать R0/R5/R10 на выбор

Создаем RAID 10
`mdadm --create /dev/md0 -l 10 -n 10 /dev/sd{b,c,d,e,f,g,h,i,j,k}`

```
# cat /proc/mdstat
Personalities : [raid10] 
md0 : active raid10 sdk[9] sdj[8] sdi[7] sdh[6] sdg[5] sdf[4] sde[3] sdd[2] sdc[1] sdb[0]
      1274880 blocks super 1.2 512K chunks 2 near-copies [10/10] [UUUUUUUUUU]
      
unused devices: <none>
```

```
# mdadm -D /dev/md0
/dev/md0:
           Version : 1.2
     Creation Time : Tue May  1 09:41:27 2018
        Raid Level : raid10
        Array Size : 1274880 (1245.00 MiB 1305.48 MB)
     Used Dev Size : 254976 (249.00 MiB 261.10 MB)
      Raid Devices : 10
     Total Devices : 10
       Persistence : Superblock is persistent

       Update Time : Tue May  1 09:41:38 2018
             State : clean 
    Active Devices : 10
   Working Devices : 10
    Failed Devices : 0
     Spare Devices : 0

            Layout : near=2
        Chunk Size : 512K

Consistency Policy : resync

              Name : otuslinux:0  (local to host otuslinux)
              UUID : d8ffe578:b2aea5e7:80f3fd28:967057d4
            Events : 17

    Number   Major   Minor   RaidDevice State
       0       8       16        0      active sync set-A   /dev/sdb
       1       8       32        1      active sync set-B   /dev/sdc
       2       8       48        2      active sync set-A   /dev/sdd
       3       8       64        3      active sync set-B   /dev/sde
       4       8       80        4      active sync set-A   /dev/sdf
       5       8       96        5      active sync set-B   /dev/sdg
       6       8      112        6      active sync set-A   /dev/sdh
       7       8      128        7      active sync set-B   /dev/sdi
       8       8      144        8      active sync set-A   /dev/sdj
       9       8      160        9      active sync set-B   /dev/sdk
```

Создаем партицию

`# gdisk /dev/md0`

```
Number  Start (sector)    End (sector)  Size       Code  Name
   1            2048         2549726   1.2 GiB     8300  Linux filesystem
```

Создаем файловую систему

`# mkfs.ext4 /dev/md0p1`

Монтируем

`# mount /dev/md0p1 /mnt`

```
# df -h
Filesystem                       Size  Used Avail Use% Mounted on
/dev/mapper/VolGroup00-LogVol00   38G  3.2G   35G   9% /
devtmpfs                         488M     0  488M   0% /dev
tmpfs                            497M     0  497M   0% /dev/shm
tmpfs                            497M  6.7M  490M   2% /run
tmpfs                            497M     0  497M   0% /sys/fs/cgroup
/dev/sda2                       1014M   63M  952M   7% /boot
tmpfs                            100M     0  100M   0% /run/user/1000
/dev/md0p1                       1.2G  3.7M  1.1G   1% /mnt
```

Для проверки работы копируем туда MBR

`# dd if=/dev/sda of=/mnt/sda.mbr bs=512 count=1`

```
# ls -l /mnt
total 20
drwx------. 2 root root 16384 May  1 10:01 lost+found
-rw-r--r--. 1 root root   512 May  1 10:05 sda.mbr
```

### сломать/починить raid

Фейлим один диск

`# mdadm -f /dev/md0 /dev/sdc`

```
# cat /proc/mdstat
Personalities : [raid10] 
md0 : active raid10 sdk[9] sdj[8] sdi[7] sdh[6] sdg[5] sdf[4] sde[3] sdd[2] sdc[1](F) sdb[0]
      1274880 blocks super 1.2 512K chunks 2 near-copies [10/9] [U_UUUUUUUU]
      
unused devices: <none>
```

```
mdadm -D /dev/md0
/dev/md0:
           Version : 1.2
     Creation Time : Tue May  1 09:41:27 2018
        Raid Level : raid10
        Array Size : 1274880 (1245.00 MiB 1305.48 MB)
     Used Dev Size : 254976 (249.00 MiB 261.10 MB)
      Raid Devices : 10
     Total Devices : 10
       Persistence : Superblock is persistent

       Update Time : Tue May  1 10:09:03 2018
             State : clean, degraded 
    Active Devices : 9
   Working Devices : 9
    Failed Devices : 1
     Spare Devices : 0

            Layout : near=2
        Chunk Size : 512K

Consistency Policy : resync

              Name : otuslinux:0  (local to host otuslinux)
              UUID : d8ffe578:b2aea5e7:80f3fd28:967057d4
            Events : 19

    Number   Major   Minor   RaidDevice State
       0       8       16        0      active sync set-A   /dev/sdb
       -       0        0        1      removed
       2       8       48        2      active sync set-A   /dev/sdd
       3       8       64        3      active sync set-B   /dev/sde
       4       8       80        4      active sync set-A   /dev/sdf
       5       8       96        5      active sync set-B   /dev/sdg
       6       8      112        6      active sync set-A   /dev/sdh
       7       8      128        7      active sync set-B   /dev/sdi
       8       8      144        8      active sync set-A   /dev/sdj
       9       8      160        9      active sync set-B   /dev/sdk

       1       8       32        -      faulty   /dev/sdc
```

Удаляем диск

`# mdadm -r /dev/md0 /dev/sdc`

Добавляем диск

`# mdadm -a /dev/md0 /dev/sdc` 


Recovering:
```
# mdadm -D /dev/md0
/dev/md0:
           Version : 1.2
     Creation Time : Tue May  1 09:41:27 2018
        Raid Level : raid10
        Array Size : 1274880 (1245.00 MiB 1305.48 MB)
     Used Dev Size : 254976 (249.00 MiB 261.10 MB)
      Raid Devices : 10
     Total Devices : 10
       Persistence : Superblock is persistent

       Update Time : Tue May  1 10:16:18 2018
             State : clean, degraded, recovering 
    Active Devices : 9
   Working Devices : 10
    Failed Devices : 0
     Spare Devices : 1

            Layout : near=2
        Chunk Size : 512K

Consistency Policy : resync

    Rebuild Status : 17% complete

              Name : otuslinux:0  (local to host otuslinux)
              UUID : d8ffe578:b2aea5e7:80f3fd28:967057d4
            Events : 24

    Number   Major   Minor   RaidDevice State
       0       8       16        0      active sync set-A   /dev/sdb
      10       8       32        1      spare rebuilding   /dev/sdc
       2       8       48        2      active sync set-A   /dev/sdd
       3       8       64        3      active sync set-B   /dev/sde
       4       8       80        4      active sync set-A   /dev/sdf
       5       8       96        5      active sync set-B   /dev/sdg
       6       8      112        6      active sync set-A   /dev/sdh
       7       8      128        7      active sync set-B   /dev/sdi
       8       8      144        8      active sync set-A   /dev/sdj
       9       8      160        9      active sync set-B   /dev/sdk
```

Recovering complete:
```
# mdadm -D /dev/md0
/dev/md0:
           Version : 1.2
     Creation Time : Tue May  1 09:41:27 2018
        Raid Level : raid10
        Array Size : 1274880 (1245.00 MiB 1305.48 MB)
     Used Dev Size : 254976 (249.00 MiB 261.10 MB)
      Raid Devices : 10
     Total Devices : 10
       Persistence : Superblock is persistent

       Update Time : Tue May  1 10:16:29 2018
             State : clean 
    Active Devices : 10
   Working Devices : 10
    Failed Devices : 0
     Spare Devices : 0

            Layout : near=2
        Chunk Size : 512K

Consistency Policy : resync

              Name : otuslinux:0  (local to host otuslinux)
              UUID : d8ffe578:b2aea5e7:80f3fd28:967057d4
            Events : 39

    Number   Major   Minor   RaidDevice State
       0       8       16        0      active sync set-A   /dev/sdb
      10       8       32        1      active sync set-B   /dev/sdc
       2       8       48        2      active sync set-A   /dev/sdd
       3       8       64        3      active sync set-B   /dev/sde
       4       8       80        4      active sync set-A   /dev/sdf
       5       8       96        5      active sync set-B   /dev/sdg
       6       8      112        6      active sync set-A   /dev/sdh
       7       8      128        7      active sync set-B   /dev/sdi
       8       8      144        8      active sync set-A   /dev/sdj
       9       8      160        9      active sync set-B   /dev/sdk
```

```
# cat /proc/mdstat
Personalities : [raid10] 
md0 : active raid10 sdc[10] sdk[9] sdj[8] sdi[7] sdh[6] sdg[5] sdf[4] sde[3] sdd[2] sdb[0]
      1274880 blocks super 1.2 512K chunks 2 near-copies [10/10] [UUUUUUUUUU]
      
unused devices: <none>
```

`# dmesg`

```
[ 3194.701318] md/raid10:md0: not clean -- starting background reconstruction
[ 3194.701333] md/raid10:md0: active with 10 out of 10 devices
[ 3194.701409] md0: detected capacity change from 0 to 1305477120
[ 3194.724003] md: resync of RAID array md0
[ 3204.505694] md: md0: resync done.
[ 4263.712509]  md0: p1
[ 4264.736745]  md0: p1
[ 4463.244785] EXT4-fs (md0p1): mounted filesystem with ordered data mode. Opts: (null)
[ 4850.226235] md/raid10:md0: Disk failure on sdc, disabling device.
md/raid10:md0: Operation continuing on 9 devices.
[ 5283.337423] md: recovery of RAID array md0
[ 5294.680272] md: md0: recovery done.
```

При перезагрузке RAID собрался сам
В руководствах пишут, что для этого надо добавлять информацию о массиве в конфиг:
`# mdadm --detail --scan >> /etc/mdadm/mdadm.conf`
Но, как я понял, используются Persistent Superblocks (можем видеть, что у нас `Persistence : Superblock is persistent`), метаданные пишутся прямо в суперблок, массив можно собрать раньше, чем смонтирована файловая система и прочитан `mdadm.conf`. 
Итого, `mdadm.conf` отсутствует, при загрузке массив собирается:
`dmesg`:
```
[   27.686494] md/raid10:md0: active with 10 out of 10 devices
[   27.686580] md0: detected capacity change from 0 to 1305477120
```


### Пробуем собрать RAID 6 + 0

Удаляем старый массив
`umount /mnt`
`mdadm -S /dev/md0`
`mdadm --zero-superblock /dev/sd{b,c,d,e,f,g,h,i,j,k}`

Создаем два RAID 6
```
# mdadm --create /dev/md0 -l 6 -n 5 /dev/sd{b,c,d,e,f}
# mdadm --create /dev/md1 -l 6 -n 5 /dev/sd{g,h,i,j,k}

Создам RAID 0 из двух RAID 6
# mdadm --create /dev/md2 -l 0 -n 2 /dev/md{0,1}
```

```
# cat /proc/mdstat
Personalities : [raid6] [raid5] [raid4] [raid0]
md2 : active raid0 md1[1] md0[0]
      1527808 blocks super 1.2 512k chunks

md1 : active raid6 sdk[4] sdj[3] sdi[2] sdh[1] sdg[0]
      764928 blocks super 1.2 level 6, 512k chunk, algorithm 2 [5/5] [UUUUU]

md0 : active raid6 sdf[4] sde[3] sdd[2] sdc[1] sdb[0]
      764928 blocks super 1.2 level 6, 512k chunk, algorithm 2 [5/5] [UUUUU]

unused devices: <none>
```
```
# lsblk
NAME                    MAJ:MIN RM  SIZE RO TYPE  MOUNTPOINT
sda                       8:0    0   40G  0 disk
├─sda1                    8:1    0    1M  0 part
├─sda2                    8:2    0    1G  0 part  /boot
└─sda3                    8:3    0   39G  0 part
  ├─VolGroup00-LogVol00 253:0    0 37.5G  0 lvm   /
  └─VolGroup00-LogVol01 253:1    0  1.5G  0 lvm   [SWAP]
sdb                       8:16   0  250M  0 disk
└─md0                     9:0    0  747M  0 raid6
  └─md2                   9:2    0  1.5G  0 raid0
sdc                       8:32   0  250M  0 disk
└─md0                     9:0    0  747M  0 raid6
  └─md2                   9:2    0  1.5G  0 raid0
sdd                       8:48   0  250M  0 disk
└─md0                     9:0    0  747M  0 raid6
  └─md2                   9:2    0  1.5G  0 raid0
sde                       8:64   0  250M  0 disk
└─md0                     9:0    0  747M  0 raid6
  └─md2                   9:2    0  1.5G  0 raid0
sdf                       8:80   0  250M  0 disk
└─md0                     9:0    0  747M  0 raid6
  └─md2                   9:2    0  1.5G  0 raid0
sdg                       8:96   0  250M  0 disk
└─md1                     9:1    0  747M  0 raid6
  └─md2                   9:2    0  1.5G  0 raid0
sdh                       8:112  0  250M  0 disk
└─md1                     9:1    0  747M  0 raid6
  └─md2                   9:2    0  1.5G  0 raid0
sdi                       8:128  0  250M  0 disk
└─md1                     9:1    0  747M  0 raid6
  └─md2                   9:2    0  1.5G  0 raid0
sdj                       8:144  0  250M  0 disk
└─md1                     9:1    0  747M  0 raid6
  └─md2                   9:2    0  1.5G  0 raid0
sdk                       8:160  0  250M  0 disk
└─md1                     9:1    0  747M  0 raid6
  └─md2                   9:2    0  1.5G  0 raid0
```

Создаем раздел
`# parted -s /dev/md2 mklabel gpt`
`# parted -s /dev/md2 mkpart primary ext4 2048s 100M`
`Warning: The resulting partition is not properly aligned for best performance`
Выравнивание оказалось неправильным. На RAID1 начало с 2048s было ок. 

Считаем, сколько нужно для этого массива
```
# cat /sys/block/md2/queue/optimal_io_size
3145728
# cat /sys/block/md2/queue/minimum_io_size
524288
# cat /sys/block/md2/alignment_offset
0
# cat /sys/block/md2/queue/physical_block_size
512
```
Add optimal_io_size to alignment_offset and divide the result by physical_block_size. (3145728 + 0) / 512 = 6144

`parted -s /dev/md2 mkpart primary ext4 6144s 100M`
```
(parted) align-check optimal 1
1 aligned
```

Провряем, что работает
`# mkfs.ext4 /dev/md2p1`
`# mount /dev/md2p1 /mnt`
`# dd if=/dev/sda of=/mnt/sda.mbr bs=512 count=1`
`# ls -l /mnt`

Пофейлить диск в mdadm можно только для md0 и md1. md2 про них ничего не знает
`# mdadm -f /dev/md0 /dev/sdc`

Видим сбой в RAID 6, у RAID 0 типа все нормально.
```
# cat /proc/mdstat
Personalities : [raid6] [raid5] [raid4] [raid0]
md2 : active raid0 md1[1] md0[0]
      1527808 blocks super 1.2 512k chunks

md1 : active raid6 sdk[4] sdj[3] sdi[2] sdh[1] sdg[0]
      764928 blocks super 1.2 level 6, 512k chunk, algorithm 2 [5/5] [UUUUU]

md0 : active raid6 sdf[4] sde[3] sdd[2] sdc[1](F) sdb[0]
      764928 blocks super 1.2 level 6, 512k chunk, algorithm 2 [5/4] [U_UUU]

unused devices: <none>
```
```
# mdadm -D /dev/md2
/dev/md2:
           Version : 1.2
     Creation Time : Fri May  4 11:54:56 2018
        Raid Level : raid0
        Array Size : 1527808 (1492.00 MiB 1564.48 MB)
      Raid Devices : 2
     Total Devices : 2
       Persistence : Superblock is persistent

       Update Time : Fri May  4 11:54:56 2018
             State : clean
    Active Devices : 2
   Working Devices : 2
    Failed Devices : 0
     Spare Devices : 0

        Chunk Size : 512K

Consistency Policy : none

              Name : otuslinux:2  (local to host otuslinux)
              UUID : fb5ed678:65ed719f:609b30f3:2bbbb9ba
            Events : 0

    Number   Major   Minor   RaidDevice State
       0       9        0        0      active sync   /dev/md0
       1       9        1        1      active sync   /dev/md1
```
Чиним md0, все ок


### прописать собранный рейд в конф, чтобы рейд собирался при загрузке
### создать GPT раздел и 5 партиций

Добавляем в Vagranfile в shell provision
```
              mdadm --create /dev/md0 -l 10 -n 10 /dev/sd{b,c,d,e,f,g,h,i,j,k}
              parted -s /dev/md0 mklabel gpt
              parted -s /dev/md0 mkpart primary ext4 2048s 100M
              parted -s /dev/md0 mkpart primary ext4 100M 200M
              parted -s /dev/md0 mkpart primary ext4 200M 300M
              parted -s /dev/md0 mkpart primary ext4 300M 400M
              parted -s /dev/md0 mkpart primary ext4 400M 100%
```

Вроде нормально: 
```
# lsblk
NAME                    MAJ:MIN RM   SIZE RO TYPE   MOUNTPOINT
sda                       8:0    0    40G  0 disk
├─sda1                    8:1    0     1M  0 part
├─sda2                    8:2    0     1G  0 part   /boot
└─sda3                    8:3    0    39G  0 part
  ├─VolGroup00-LogVol00 253:0    0  37.5G  0 lvm    /
  └─VolGroup00-LogVol01 253:1    0   1.5G  0 lvm    [SWAP]
sdb                       8:16   0   250M  0 disk
└─md0                     9:0    0   1.2G  0 raid10
  ├─md0p1               259:4    0  94.4M  0 md
  ├─md0p2               259:5    0  95.4M  0 md
  ├─md0p3               259:6    0  95.4M  0 md
  ├─md0p4               259:7    0  95.4M  0 md
  └─md0p5               259:8    0 863.5M  0 md

```
Для остальных дисков то же самое


### в качестве проверки принимаются - измененный Vagrantfile, скрипт для создания рейда, конф для автосборки рейда при загрузке

- Vagrantfile в репозитории
- скрипт для создания рейда в Vagrantfile
- конф для автосборки рейда при загрузке
- конф для автосборки рейда при загрузке `# mdadm --detail --scan`, записан в суперблок
