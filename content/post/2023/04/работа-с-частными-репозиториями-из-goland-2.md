---
title: "Работа с частными репозиториями из Goland (Golang) 2"
date: "2023-04-27"
categories: 
  - "golang"
tags: 
  - "gitlab"
  - "goland"
  - "golang"
  - "https"
  - "import"
  - "modules"
---
Golang при работе с внешними зависимостями по умолчанию работает с `https` протколом. Поэтому с настройками по умолчанию возникает ошибка ниже:

```go
go: gitlab.space.team/dev/spacevm-go@v0.0.0-20230427205557-c59c016940d9: invalid version: git ls-remote -q origin in 
/home/dp/Code/golang/go/pkg/mod/cache/vcs/9d6ac92dae2cb8c34a30234f55abdae32ec5751ebe1104a5d00259017d7295a9: exit status 128:
	fatal: could not read Username for 'http://gitlab.space.team':
	terminal prompts disabled Confirm the import path was entered correctly.
If this is a private repository, see https://golang.org/doc/faq#git_https for additional information.
```

<!--more-->

Что бы заставить его использовать `http` при работе частными репозиториями неободимо прописать авторизацию через файл `$HOME/.netrc`

```bash
machine gitlab.space.team login USERNAME password PASSWORD(APIKEY)
```

При возникновении следующей ошибки:

```go
go: terraform-provider-spacevm imports
	gitlab.space.team/dev/spacevm-go/api: gitlab.space.team/dev/spacevm-go@v0.0.0-20230302093536-f65e5b53e820: parsing go.mod:
	module declares its path as: spacevm-go
	        but was required as: gitlab.space.team/dev/spacevm-go

```

необходимо в самописном модуле который импортируем и который расположен в приватном репозитории прописать правильное именование.

Вместо `module spacevm-go` необходимо писать полный путь до приватного модуля `module gitlab.space.team/dev/spacevm-go`.

![](/images/2023/04/gomod.png)

