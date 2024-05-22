---
title: "Работа с частными репозиториями из Goland (Golang) 1"
date: "2022-01-13T06:48:00+03:00"
categories:
  - "golang"
tags:
  - "gitlab"
  - "goland"
  - "https"
  - "import go modules"
---

**Golang** и линейка продуктов **JetBrains**, куда входят **Goland** и **Pycharm**, ориентированны на работу с внешними сервисами, такими как **Github**.
**Github** работает по `https` протоколу, для менеджера пакетов **Golang** необходима система контроля версий, поддерживающая `https` протокол, иначе он не захочет качать пакеты.

<!--more-->

Во множестве организаций использутся внутренние репозитории кода. У нас код хостится на внутреннем сервере **Gitlab**.
При этом `https` не используется, используется `http`. При импорте модулей в пакет при разработке на **Golang** возникает проблема:

![](/images/2022/01/golang_https1.png)

При попытке импорта, **Goland** пишет:

```go
go: finding module for package gitlab.bazalt.team/dev/veil-api-client-go/veil
terraform-provider-veil imports
gitlab.bazalt.team/dev/veil-api-client-go/veil: cannot find module providing package gitlab.bazalt.team/dev/veil-api-client-go/veil: unrecognized import path "gitlab.bazalt.team/dev/veil-api-client-go/veil": https fetch: Get "https://gitlab.bazalt.team/dev/veil-api-client-go/veil?go-get=1": dial tcp 192.168.14.215:443: connect: connection refused
```

Что делал что бы побороть ошибку:

- запилил самоподписанные сертфикаты для **Gitlab**, не помогло:

```go
go: finding module for package gitlab.bazalt.team/dev/veil-api-client-go/veil
terraform-provider-veil/veil imports
gitlab.bazalt.team/dev/veil-api-client-go/veil: cannot find module providing package gitlab.bazalt.team/dev/veil-api-client-go/veil: unrecognized import path "gitlab.bazalt.team/dev/veil-api-client-go/veil": https fetch: Get "https://gitlab.bazalt.team/dev/veil-api-client-go/veil?go-get=1": x509: certificate signed by unknown authority

```

- экспериментировал с `git` на локальном хосте как [здесь](https://stackoverflow.com/questions/29707689/how-to-use-go-with-a-private-gitlab-repo), не помогло:

- поспал, начал читать разное про модули, в итоге набрел на ["Go Modules Reference"](https://go.dev/ref/mod#environment-variables), помогло...

В настройках проекта Goland устанавливаем переменные с исключениями из внутренних серверов и удаляем старые версии модулей в консоли:

![](/images/2022/01/golang_settings.png)

```bash
go clean -modcache
```

Может потребоваться перезапуск Goland. Модули нормально импортировались:

![](/images/2022/01/golang_https2.png)
