# Homework 4

Работа с загрузчиком
1. Попасть в систему без пароля несколькими способами
2. Установить систему с LVM, после чего переименовать VG
3. Добавить модуль в initrd

4(*). Сконфигурировать систему без отдельного раздела с /boot, а только с LVM
Репозиторий с пропатченым grub: https://yum.rumyantsev.com/centos/7/x86_64/
PV необходимо инициализировать с параметром --bootloaderareasize 1m


### Попасть в систему без пароля несколькими способами

1.
- Грузимся с LiveCD
- Выбираем `Troubleshooting`
- `Rescue a CentOS System`
- `Will attempt to find the installation and mount it to /mnt/sysimage`
- `Continue`
- Mounted
- `# chroot /mnt/sysimage`
- `# passwd`
- `# rm -f  /.autorelabel`
- `# exit`
- `# exit`

2.
- `e`
- Убираем `rhgb` и `quiet`, если есть, добавляем в конец `rd.break enforcing=0`
- Ctrl-x для продолжения загрузки
- `# mount -o remount,rw /sysroot`
- `# chroot /sysroot`
- `# passwd`
- `# touch /.autorelabel`
- `# mount -o remount,ro /`
- `# exit` 
- `# exit`

3.
- `e`
- `ro` меняем на `rw init=sysroot//bin/sh`
- `# mount -o remount,rw /sysroot`
- `# chroot /sysroot`
- `passwd`
- `# touch /.autorelabel`
- `# mount -o remount,ro /`
- `# exit` 
- `# exit`


###  Установить систему с LVM, после чего переименовать VG

- Установил OS с `/` и `/home` на LVM, название VG: `centos`
- Преименовываем VG:
`# vgrename -v centos newvg`
- Меняем название VG в конфигах `grub` и `fstab`:
`# vi /etc/fstab`, `vi /etc/default/grub`, `vi /boot/grub2/grub.cfg`. Везде меняем `centos` -> `newvg`
Без правки grub.cfg не получилось, хотя это вроде неправильно
- Генерим новый initramfs
- `# dracut -f /boot/initramfs-$(uname -r).img $(uname -r)`
- `# reboot`
Загрузилось нормально на новой VG


### Добавить модуль в initrd

- Создаем скрипты модуля и установки
`# mkdir /usr/lib/dracut/modules.d/01test`
`# vi /usr/lib/dracut/modules.d/01test/module-setup.sh`
`# vi /usr/lib/dracut/modules.d/01test/test.sh`
`# chmod +x /usr/lib/dracut/modules.d/01test/test.sh
`# chmod + /usr/lib/dracut/modules.d/01test/module_setup.sh
- Генерим образ
`# dracut -f -v /boot/initramfs-$(uname -r).img $(uname -r)
`*** Including module: test ***`
- Провряем наличие модуля и образе
```
# lsinitrd -m /boot/initramfs-$(uname -r).img
Image: /boot/initramfs-3.10.0-862.2.3.el7.x86_64.img: 15M
========================================================================
Version: dracut-033-535.el7

dracut modules:
bash
test
```
