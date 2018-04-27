# Homework 1

### Сборка ядра

Версия ядра: linux-4.16.4

`yum.log`:

```
Apr 25 09:50:34 Installed: wget-1.14-15.el7_4.1.x86_64
```

```
Apr 25 10:13:54 Installed: mpfr-3.1.1-4.el7.x86_64
Apr 25 10:13:54 Installed: libmpc-1.0.1-3.el7.x86_64
Apr 25 10:13:58 Installed: cpp-4.8.5-16.el7_4.2.x86_64
Apr 25 10:14:02 Installed: kernel-headers-3.10.0-693.21.1.el7.x86_64
Apr 25 10:14:05 Installed: glibc-headers-2.17-196.el7_4.2.x86_64
Apr 25 10:14:06 Installed: glibc-devel-2.17-196.el7_4.2.x86_64
Apr 25 10:14:14 Installed: gcc-4.8.5-16.el7_4.2.x86_64
Apr 25 10:16:28 Installed: m4-1.4.16-10.el7.x86_64
Apr 25 10:16:28 Installed: bison-3.0.4-1.el7.x86_64
Apr 25 10:16:49 Installed: flex-2.5.37-3.el7.x86_64
Apr 25 10:27:15 Installed: zlib-devel-1.2.7-17.el7.x86_64
Apr 25 10:27:15 Installed: elfutils-libelf-devel-0.168-8.el7.x86_64
Apr 25 10:30:52 Installed: libkadm5-1.15.1-8.el7.x86_64
Apr 25 10:30:53 Installed: libsepol-devel-2.5-6.el7.x86_64
Apr 25 10:30:54 Installed: pcre-devel-8.32-17.el7.x86_64
Apr 25 10:30:55 Installed: libselinux-devel-2.5-11.el7.x86_64
Apr 25 10:30:55 Installed: libcom_err-devel-1.42.9-10.el7.x86_64
Apr 25 10:30:55 Installed: libverto-devel-0.2.5-4.el7.x86_64
Apr 25 10:30:55 Installed: keyutils-libs-devel-1.5.8-3.el7.x86_64
Apr 25 10:30:56 Installed: krb5-devel-1.15.1-8.el7.x86_64
Apr 25 10:31:02 Installed: 1:openssl-devel-1.0.2k-8.el7.x86_64
Apr 25 10:32:15 Installed: bc-1.06.95-13.el7.x86_64
```

```
cp /boot/config* .config
make oldconfig
make -j4
make modules_install
make install
vi /etc/default/grub
GRUB_DEFAULT=0
grub2-mkconfig -o /boot/grub2/grub.cfg
```
```
$ uname -a
Linux otuslinux 4.16.4 #1 SMP Wed Apr 25 10:32:45 UTC 2018 x86_64 x86_64 x86_64 GNU/Linux
```
