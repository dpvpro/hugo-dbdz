---
title: "Работа с частными репозиториями в Golang - Goinsecure"
date: "2022-01-13T06:48:00+03:00"
description: Опция goinsecure в Golang.
categories:
  - "golang"
tags:
  - "go modules import"
  - "golang gitlab"
  - "golang private repo"
  - "golang частные репозитории"
  - "goinsecure"
---

Язык программирования Golang и линейка продуктов JetBrains, куда входят Goland и Pycharm, ориентированны на работу с внешними сервисами, такими как Github.
Github использует `https` протокол.  В менеджере пакетов Golang  по умолчанию настроена работа с использованием `https` протокола.

<!--more-->

Во множестве организаций использутся внутренние репозитории кода. У нас код хостится на внутреннем сервере Gitlab.
При этом `https` не используется, используется `http`. При импорте модулей в пакет при разработке на Golang возникает проблема:

![](/images/2022/01/golang_https1.png)

При попытке импорта, Goland пишет:

```go
go: finding module for package gitlab.bazalt.team/dev/veil-api-client-go/veil
terraform-provider-veil imports
gitlab.bazalt.team/dev/veil-api-client-go/veil: cannot find module providing package gitlab.bazalt.team/dev/veil-api-client-go/veil: unrecognized import path "gitlab.bazalt.team/dev/veil-api-client-go/veil": https fetch: Get "https://gitlab.bazalt.team/dev/veil-api-client-go/veil?go-get=1": dial tcp 192.168.14.215:443: connect: connection refused
```

Что делал что бы побороть ошибку:

- запилил самоподписанные сертфикаты для Gitlab, не помогло:

```go
go: finding module for package gitlab.bazalt.team/dev/veil-api-client-go/veil
terraform-provider-veil/veil imports
gitlab.bazalt.team/dev/veil-api-client-go/veil: cannot find module providing package gitlab.bazalt.team/dev/veil-api-client-go/veil: unrecognized import path "gitlab.bazalt.team/dev/veil-api-client-go/veil": https fetch: Get "https://gitlab.bazalt.team/dev/veil-api-client-go/veil?go-get=1": x509: certificate signed by unknown authority

```

- экспериментировал с `git` на локальном хосте как [здесь](https://stackoverflow.com/questions/29707689/how-to-use-go-with-a-private-gitlab-repo), не помогло:

- поспал, начал читать разное про модули, набрел на ["Go Modules Reference"](https://go.dev/ref/mod#environment-variables), в итоге помогло...

В настройках проекта Goland устанавливаем переменные с исключениями из внутренних серверов:

![](/images/2022/01/golang_settings.png)

и удаляем старые версии модулей в консоли:

```bash
go clean -modcache
```

Может потребоваться перезапуск Goland. Модули нормально импортировались:

![](/images/2022/01/golang_https2.png)
