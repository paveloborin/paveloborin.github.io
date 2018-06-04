---
title: "Инструменты для форматирования и анализа кода в Go"
date: 2018-06-04T02:43:57+05:00
draft: false
tags: ["go", "tools", "goimports", "errcheck", "gometalinter"]
---


## Форматирование импортов

Использование:

<code class="hljs haskell">
$ goimports -d -w $(find . -type f -name '*.go' -not -path "./vendor/*")
</code>

Установка:
<code class="hljs haskell">
$ go get golang.org/x/tools/cmd/goimports
</code>

## Автоформатирование кода

Использование:

<code class="hljs haskell">
$ go fmt ./…
</code>

## Анализ корректной обработки ошибок

Использование:

<code class="hljs haskell">
$ errcheck ./…
</code>

Установка:

<code class="hljs haskell">
$ go get -u github.com/kisielk/errcheck
</code>

## Анализ кода по множеству параметров

Использование:

<code class="hljs haskell">
$ gometalinter ./… --exclude=vendor
</code>

Установка:

<code class="hljs haskell">
$ go get -u gopkg.in/alecthomas/gometalinter.v2
<br/>
$ gometalinter --install
</code>

