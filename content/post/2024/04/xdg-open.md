---
title: "Отладка неправильного поведения утилиты xdg-open"
date: 2024-04-22T22:01:55+03:00
toc: false
comments: true
categories:
- linux
- opensource
tags:
- xdg-open
- troubleshooting
---

Проблема началась с того что после монтирования контейнера Veracrypt, перестал открываться нормально файловый менеджер. Он открывался, но параллельно открывался браузер. Ранее такого поведения не было.

<!--more-->

Я начал расследование этой ситуации. Помня, что уже когда то возился утилитами xdg-utils, я начал пробовать смотреть как настроены ассоциации через утилиты cli:

```bash
xdg-open ~/Downloads/
xdg-mime query filetype ~/Downloads/
xdg-mime default thunar.desktop inode/directory
```

Натолкнулся на то что утилита xdg-open открывает путь до точки монтирования НЕ корректно:

`xdg-open /run/media/dp/test`

Воспроизвелась ситуация когда открывается одновременно браузер и файловый менеджер.

Начал смотреть в эту сторону xdg-open.  После того как я сделал изменения зафиксированные [здесь](https://gitlab.freedesktop.org/xdg/xdg-utils/-/merge_requests/110) , поведение была исправлено.

Но ментейнер проекта отказался принимать изменения пока не докопается до причины такого поведения. Я с ним согласен, так как это правильно.

Побочным эффектом явилось то, что ментейнер предложил обходное решение в виде в файл конфигурации ассоциаций `~/.config/mimeapps.list `

```bash
 inode/directory=nemo.desktop
 inode/mount-point=nemo.desktop
```
Здесь можно прописать ваши ассоциации.

Причиной проблем явилось баг в зависимом пакете `perl-file-mimeinfo` в который входит утилита `mimetype`. Утилита `mimetype` используется в `xdg-open`. На момент разбора, в коде  `mimetype` производился безусловный выход с кодом [ошибки](https://github.com/mbeijen/File-MimeInfo/issues/54).


Было сформирован [MR](https://github.com/mbeijen/File-MimeInfo/pull/55) c исправлением. Ждем когда исправления будут доставлены в дистрибутивы.