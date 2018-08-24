---
title: "Некоторые команды Кубернетиса"
date: 2018-08-23T12:30:57+05:00
draft: false
tags: ["kubernetes"]
---
У меня на рабочем столе лежит блокнот с наиболее часто используемыми командами Кубернетиса, который я уже редко открываю.
Блокнот давно пора выбросить, а данные "оцифровать" - кому-нибудь еще могут пригодиться.
Сперва хотел сделать из заметок некий справочник, но команды Кубренетиса такие очевидные, что на мой взгляд дополнительных пояснений не требуют. 

INSTALL MAC

curl -LO https://storage.googleapis.com/kubernetes-release/release/`curl -s https://storage.googleapis.com/kubernetes-release/release/stable.txt`/bin/darwin/amd64/kubectl

chmod +x ./kubectl

sudo mv ./kubectl /usr/local/bin/kubectl


VERSION, HELP

- kubectl version 

- kubectl help


PROXY

- kubectl proxy

CONFIG

Установка контекста

- kubectl config set-context my-context --namespace=mystuff

- kubectl config use-context my-context



LABEL

- kubectl label pods bar color=red

DEBUGGING

- kubectl logs pod-name

- kubectl exec -it pod-name --bash

- kubectl cp pod-name:/path/to/remote/file /path/to/local/file


DELETE

- kubectl delete resource-name obj-name

- kubectl delete -f obj.yaml 

EDIT

- kubectl edit resource-name obj-name

- kubectl edit deployment/alpaca-prod

- kubectl edit service alpaca-prod


DESCRIBE

- kubectl describe service alpaca-prod

- kubectl describe nodes node-1

GET

- kubectl get pods -o wide --show-labels

- kubectl get deployments --namespace=kube-system kybe-dns

- kubectl get services --namespace=kube-system kybe-dns

- kubectl get componentstatuses

- kubectl get nodes

- kubectl get services -o wide

- kubectl get pods -l app=redis -o jsonpath='{.items[0].metadata.name}'

- kubectl get pods my-pod -o jsonpath --template={.status.podIP}

- kubectl get secrets

- kubectl get configmaps

- kubectl get deployments

- kubectl get replicasets -o wide


APPLY

- kubectl apply -f obj.yaml

- kubectl apply -f nginx-deployment.yaml

ROLLOUT

- kubectl rollout history deployment nginx


AUTOSCALE

kubectl autoscale deployment --min=1 --max=5 --cpu-percent=40 resource_name

