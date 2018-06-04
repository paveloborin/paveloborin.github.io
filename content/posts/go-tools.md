---
title: "Инструменты для форматирования и анализа кода в Go"
date: 2018-06-04T02:43:57+05:00
draft: false
tags: ["go", "tools"]
---


## Форматирование импортов

Использование:

goimports -d -w $(find . -type f -name '*.go' -not -path "./vendor/*")

Установка:

go get golang.org/x/tools/cmd/goimports

## Автоформатирование кода

Использование:

go fmt ./…

## Анализ корректной обработки ошибок

Использование:

errcheck ./…

Установка:

go get -u github.com/kisielk/errcheck

## Анализ кода по множеству параметров:

Использование:

gometalinter ./… --exclude=vendor

Установка:

go get -u gopkg.in/alecthomas/gometalinter.v2

gometalinter --install