---
title: "Как прокинуть доступ из контейнера к внешним сервисам с самоподписаными сертификатами"
date: 2018-07-21T12:30:57+05:00
draft: false
tags: ["docker"]
---
**Задача:**

Нужно сделать запрос из апи, развернутой в контейнере, к апи, стоящей за доменом с самоподписанным сертификатом.

Обычно для того чтобы зайти локально из браузера на такой сайт необходимо принять сертификат.
При попытке же вызвать такой сервис из докера будет ошибка: _«x509: certificate signed by unknown authority»_.

**Решение:**

Прокинуть файл сертификата в контейнер и добавить в список доверенных.

Для нужно добавить в ваш Dockerfile следующие строки:

<code class="hljs shell">
COPY tmp/Company-issuer-chain.pem /usr/local/share/ca-certificates/Company-issuer-chain-chain.crt

RUN update-ca-certificates
</code>

, где *tmp/Company-issuer-chain.pem* - путь до сертификата.

