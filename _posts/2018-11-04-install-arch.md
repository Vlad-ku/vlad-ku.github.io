---
layout: post
title: Установка Arch linux
---

В данном руководстве рассмотрим минимальную установку arch, позволяющую запустить систему.

## Настройка сети

Для настройки wi-fi необходимо сформировать файл профиля и запустить его. Для формирования файла воспользуемся утилитой `wifi-menu` и назовем новый профиль `me`.

После выбора сети и подтверждения будет сформирован файл `/etc/netctl/me`. Если подключаемся к скрытой сети, необходимо дополнить строки `ESSID` и `Hidden=yes`.

Для запуска выполняем команду:

```
netctl start profile
```

## Разметка диска
  - cfdisk /dev/sda                                         (должен быть gpt, смотреть вверху)
    - 500M  = EFI / BIOS boot
    - other = Linux filesystem
    - "yes"
  - mkfs.fat -F32 /dev/sda1
  - mkfs.ext4 /dev/sda2

## Mонтирование
  - монтируем в /mnt созданные диски                        (с созданием папки boot и др.)

## Установка системы
  - pacman -Syu                                             (обновляем ядро)
  - pacstrap /mnt base base-devel                           (установка самой системы)
  - genfstab -U /mnt >> /mnt/etc/fstab                      (файл точек монтирования систем)

## Настройка
  - arch-chroot /mnt                                        (заходим в систему без перезапуска)
  - mkinitcpio -p linux                                     (начальная файловая система)

  - pacman -S wpa_supplicant                                (для работы wifi)
  - с шага 1 файл с настройками скопировать на диск
  - netctl enable name_file                                 (настройка сети по умолчанию)

## Загрузчик
  - pacman -S grub efibootmgr
  - grub-install /dev/sda
  - grub-mkconfig -o /boot/grub/grub.cfg
