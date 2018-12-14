---
layout: post
title: Настройка Arch linux
---

Продолжаем тему установки. В первой статье мы рассмотрели минимальную установку archlinux, позволяющую запустить систему (ссылка в конце статьи). В этой - рассмотрим доведение системы до презентабильного вида.

## То, без чего невозможна настройка

```
pacman -S vim
```

## Создание пользователя

Для создания нового пользователя (не обладающего правами root) и задания паролей выполним следующие команды:

```
useradd -m -g users -G wheel -s /bin/bash vlad
passwd vlad
passwd
```

Дальнейшую настройку можно продолжить под новым пользователем. В этом случае некоторым командам может потребоваться выполнение через `sudo`.

## Настройка часового пояса и времени

```
mv /etc/localtime /etc/localtime.old
ln -s /usr/share/zoneinfo/Asia/Irkutsk /etc/localtime
hwclock --systohc --utc
```

## Настройка локали

Для настройки необходимо раскомментировать нужные строки и заново сгенерировать локали, после чего указать нужную.

```
vim /etc/locale.gen
locale-gen
echo "LANG=ru_RU.UTF-8" > /etc/locale.conf
```

## Настройка раскладки (для консоли)

Для указания шрифта и способа переключения раскладки открываем файл:

```
vim /etc/vconsole.conf
```

И указываем следующее:

```
KEYMAP=ruwin_alt_sh-UTF-8
FONT=UniCyr_8x16
```

## Настройка сетевого имени

Для указания сетевого имени выполняем:

```
echo "zalman" > /etc/hostname
```

Далее открываем файл:

```
vim /etc/hosts
```

И дописываем строчку:

```
127.0.0.1 zalman.localdomain zalman
```

## Настройка X

Настраивать будем тайловый оконный менеджер `i3wm`. Для авторизации используем `slim`. На первом шаге ставим X сервер и драйвера:

```
pacman -S xorg-server xorg-drivers
```

Вторым шагом ставим тайловый менеджер, меню запуска и окно входа:

```
pacman -S i3 dmenu slim
```

Что бы slim после удачной авторизации передавал управление дальше, в файле:

```
vim ~/.xinitrc
```

Необходимо указать:

```
exec i3
```

Далее прописываем окно авторизации в автозапуск (предварительно можно протестировать, заменив `enable` на `start`):

```
sudo systemctl enable slim
```

## Настройка раскладки (для Х org)

Если в графическом режиме переключение раскладки не работает, следует посмотреть текущие настройки:

```
setxkbmap -print -verbose 10
```

Пример ответа с рабочей системы:

```
Setting verbose level to 10
locale is C
Trying to load rules file ./rules/evdev...
Trying to load rules file /usr/share/X11/xkb/rules/evdev...
Success.
Applied rules from evdev:
rules:      evdev
model:      pc105
layout:     us,ru
variant:    ,
options:    grp:alt_shift_toggle,grp_led:scroll,grp:alt_shift_toggle,grp_led:scroll
Trying to build keymap using the following components:
keycodes:   evdev+aliases(qwerty)
types:      complete
compat:     complete+ledscroll(group_lock)
symbols:    pc+us+ru:2+inet(evdev)+group(alt_shift_toggle)
geometry:   pc(pc105)
xkb_keymap {
	xkb_keycodes  { include "evdev+aliases(qwerty)"	};
	xkb_types     { include "complete"	};
	xkb_compat    { include "complete+ledscroll(group_lock)"	};
	xkb_symbols   { include "pc+us+ru:2+inet(evdev)+group(alt_shift_toggle)"	};
	xkb_geometry  { include "pc(pc105)"	};
};
```

Следует добавить следующий код:

```
Section "InputClass"
    Identifier "system-keyboard"
	Option "XkbLayout" "us,ru"
	Option "XkbModel" "pc105"
	Option "XkbVariant" ","
	Option "XKbOptions" "grp:alt_shift_toggle,grp_led:scroll"
EndSection
```

В файл:

```
/etc/X11/xorg.conf.d/00-keyboard.conf
```

## "Полировка" системы

### Тема для окна авторизации

Окно входа по умолчанию выглядит хоть и хорошо, после консоли, но недостаточно. Установим другую тему.

```
pacman -S slim-themes archlinux-themes-slim
```

Что бы указать новую тему, откроем файл:

```
vim /etc/slim.conf
```

И изменим строчку с `current_theme` на это:

```
current_theme archlinux
```

### Тема для общего оформления системы

Для себя я выбрал тему [arc-theme](https://github.com/horst3180/Arc-theme). Для настройки нам понадобится утилита `lxappearance` и сама тема. Через утилиту переключаем тему на новую и настраиваем другие параметры, по необходимости.

```
pacman -S lxappearance arc-gtk-theme
```

### Настройка ssh

Для настройки ssh необходимо установить пакет `openssh` и набор сетевых утилит `net-tools` (туда входит ifconfig).

```
pacman -S openssh net-tools
```

Для настройки серверного режима снимаем комментарий с порта (или ставим свой порт) в файле:

```
/etc/ssh/sshd_config
```

И настраиваем демон ssh в автозапуске:

```
sudo systemctl enable sshd
```

### Настройка звука

```
pacman -S alsa-utils alsa-plugins
```

Для управления:

```
alsamixer
```

Для проверки звука:

```
speaker-test -c 2
```

## Ссылки

* [Первая статья по установке](/20181104-install-arch)
* [Репозиторий для настройки i3wm](https://github.com/vlad-ku/i3conf)
