---
title: "Переустановка WSUS с настройками по умолчанию"
date: "2015-08-04"
categories:
  - "windows"
tags:
  - "переустановка wsus"
---

<!--more-->

Переустановка **WSUS** с чистой базой данных, без данных предыдущей конфигурации.

Запустите **Powershell** как Администратор и выполните следующие команды:

```powershell
Uninstall-WindowsFeature -Name UpdateServices,Windows-Internal-Database -Restart
```

Роль удалится, сервер перезагрузится.

После перезагрузки, удалите ВСЕ из папки `C:\\Windows\\WID` (актуально для **Win 2012 R2**).

После этого запустите следующую команду в консоли **Powershell**, которая установит роль **WSUS**:

```powershell
Install-WindowsFeature UpdateServices -Restart
```

Это работает для Powershell 3 и выше.

[Источник](https://serverfault.com/questions/449914/how-to-completely-wipe-wsus-and-start-again/618951#618951?newreg=38e21e641b7942f8aa84e7476dc63703)
