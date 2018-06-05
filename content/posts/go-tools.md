---
title: "Инструменты для форматирования и анализа кода в Go"
date: 2018-06-04T02:43:57+05:00
draft: false
tags: ["go", "tools", "goimports", "errcheck", "gometalinter"]
---


## Форматирование импортов

Использование:

<code class="hljs shell">
<span class="hljs-meta">$</span> goimports -d -w $(find . -type f -name '*.go' -not -path "./vendor/*")
</code>

Установка:
<code class="hljs shell">
<span class="hljs-meta">$</span> go get golang.org/x/tools/cmd/goimports
</code>

## Автоформатирование кода

Использование:

<code class="hljs shell">
<span class="hljs-meta">$</span> go fmt <span class="hljs-string">./...</span>
</code>

## Анализ корректной обработки ошибок

Использование:

<code class="shell hljs">
<span class="hljs-meta">$</span> <span class="bash">errcheck</span> <span class="hljs-string">./...</span>
</code>

Установка:

<code class="hljs shell">
<span class="hljs-meta">$</span> go get -u github.com/kisielk/errcheck
</code>

## Анализ кода по множеству параметров

Использование:

<code class="hljs shell">
<span class="hljs-meta">$</span> gometalinter <span class="hljs-string">./...</span> --exclude=vendor
</code>

Установка:

<code class="hljs shell">
<span class="hljs-meta">$</span> go get -u gopkg.in/alecthomas/gometalinter.v2
<br/>
<span class="hljs-meta">$</span> gometalinter --install
</code>

