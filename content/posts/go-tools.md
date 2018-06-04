---
title: "Инструменты для форматирования и анализа кода в Go"
date: 2018-06-04T02:43:57+05:00
draft: false
tags: ["go", "tools", "goimports", "errcheck", "gometalinter"]
---


## Форматирование импортов

Использование:

<code class="hljs shell">
$ goimports -d -w $(find . -type f -name '*.go' -not -path "./vendor/*")
</code>

Установка:
<code class="hljs shell">
$ go get golang.org/x/tools/cmd/goimports
</code>

## Автоформатирование кода

Использование:

<code class="hljs shell">
$ go fmt ./…
</code>

## Анализ корректной обработки ошибок

Использование:

<code class="shell hljs">
<span class="hljs-meta">$</span> <span class="bash">errcheck</span> <span class="hljs-string">./...</span>
</code>

Установка:

<code class="hljs shell">
$ go get -u github.com/kisielk/errcheck
</code>

## Анализ кода по множеству параметров

Использование:

<code class="hljs shell">
$ gometalinter ./… --exclude=vendor
</code>

Установка:

<code class="hljs shell">
$ go get -u gopkg.in/alecthomas/gometalinter.v2
<br/>
$ gometalinter --install
</code>

