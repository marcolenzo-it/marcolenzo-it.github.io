---
title: 'Installa Kubernetes su Ubuntu con MicroK8s'
date: '2022-12-10'
author: marcolenzo
excerpt: 'Impariamo a installare e utilizzare Kubernetes su Ubuntu in pochissimi minuti con MicroK8s. MicroK8s è una distribuzione certificata di Kubernetes supportata da Canonical.'
layout: post
permalink: /installa-kubernetes-su-ubuntu-con-microk8s/
categories:
    - 'DevOps'
tags:
    - Kubernetes
    - Microk8s
---

In questo tutorial, ti mostro come installare Kubernetes su Ubuntu in pochissimi minuti grazie a [MicroK8s](https://microk8s.io/) di Canonical. Inoltre, impareremo e useremo i concetti di Pod, Deployment, Service e Ingress che sono alla base del successo di Kubernetes.

Kubernetes è senza dubbio la tecnologia più dirompente dell’ultimo decennio nel campo dello sviluppo software. Il movimento DevOps non sarebbe lo stesso senza Kubernetes. Per questo motivo, è molto importante conoscere almeno le basi.

## Video

Se preferisci puoi vedere questo tutorial su YouTube.

{% include embed/youtube.html id='XXAUjfKNN4E' %}

## Cos’è MicroK8s

**MicroK8s è una distribuzione certificata di Kubernetes** curata da Canonical che ha come obiettivo essere semplice da installare e gestire, e leggera per poterla utilizzare su qualunque tipo di device. Infatti, è cosi leggera che potresti installarla anche su una Raspberry PI.

Un altro punto di forza di MicroK8s è che **gira su Linux, Window, e macOS**, quindi non ci sono scuse per non provarla.

In questo articolo, installeremo MicroK8s su Ubuntu. Io userò la versione LTS più moderna che, al momento, è 22.04. Tu puoi utilizzare una versione qualunque a partire dalla 16.04.

Tutto quello di cui abbiamo bisogno è una VM, PC o Server con almeno 2 vCPU e 4GB RAM. La RAM che MicroK8s consuma è inferiore a 600MB, ma dobbiamo averne un po’ di più per poter lanciare le nostre applicazioni.

## Installazione

L’installazione consiste di una sola linea di comando perché utilizzeremo una `snap` preparata da Canonical.

```shell
sudo snap install microk8s --classic --channel=1.25
```

Con una connessione internet veloce questo comando dovrebbe completare in pochi secondi. Dopo di che, passiamo alla configurazione del gruppo `microk8s` aggiungendo il nostro utente per evitare di usare `sudo `ogni qualvolta dobbiamo invocare `microk8s`.

```shell
sudo usermod -a -G microk8s $USER<br></br>sudo chown -f -R $USER ~/.kube<br></br>su - $USER
```

Adesso controlliamo se `microk8s `è già configurato e pronto per l’uso. Utilizzando la flag `--wait-ready` possono passare alcuni secondi o minuti prima che riceviamo un output.

```shell
microk8s status --wait-ready
```

Se tutto è in ordine dovremmo vedere una schermata simile alla seguente.

![Microk8s Status](/assets/img/2022/12/microk8s-status.jpg)

## Attiviamo le addon

MicroK8s ha un sistema di addon che ci permettono di attivare delle funzionalità importanti come DNS, Ingress e Storage.

```shell
microk8s enable dns ingress storage
```

## Pro-Tip: Completamento e Alias

MicroK8s espone la Kubernetes CLI come `microk8s.kubectl`. Troppo lungo come comando. Ci creeremo un bell’alias, `k`, e attiveremo il completamento automatico, senza di cui lavorare con Kubernetes sarebbe un incubo.

```shell
echo 'source <(microk8s.kubectl completion bash)' >>~/.bashrc
echo 'alias k=microk8s.kubectl' >>~/.bashrc
echo 'complete -o default -F __start_kubectl k' >>~/.bashrc
exec bash
```

## Pods

![Kubernetes Pod](/assets/img/2022/12/pod.jpg)

I [Pod](https://kubernetes.io/docs/concepts/workloads/pods/) sono l’unità di base in Kubernetes. Rappresentano un gruppo di processi e risorse isolate che possono essere condivise da uno o più containers. Solitamente si rilascia un’applicazione per Pod dove l’applicazione è il container principale. Gli altri container presenti sono chiamati sidecar e hanno lo scopo di facilitare alcune funzioni generiche come logging, sicurezza, ecc.

Creeremo due Pod: uno con Nginx come web server; e un altro con BusyBox per potere interagire col web server.

```shell
k run pod1 --image nginx<br></br>k run pod2 --image busybox --command -- sleep infinity<br></br>k get pods -w
```

![Kubernetes Pod Running](/assets/img/2022/12/pod-running.jpg)

## Deployments

![Kubernetes Deployment](/assets/img/2022/12/deployment.jpg)

In produzione difficilmente dichiariamo Pod, ma utilizziamo invece altri concetti come il [Deployment](https://kubernetes.io/docs/concepts/workloads/controllers/deployment/). I Deployment ci permettono di definire un numero di istanze che vogliamo del nostro Pod e anche di specificare quali strategie di deployment preferiamo per aggiornare i nostri Pod. Nel nostro caso, richiederemo due Pod con Nginx web server.

```shell
k delete pod pod1<br></br>k create deployment nginx --image nginx --replicas 2<br></br>k get pods -w
```

![Kubernetes Deployment Creating](/assets/img/2022/12/deployment-creating.jpg)

## Services

![Kubernetes Service](/assets/img/2022/12/service.jpg)

Se volessimo accedere il nostro web server potremmo utilizzare l’IP del target Pod, ma sarebbe l’approccio sbagliato. In Kubernetes esponiamo le nostre applicazioni come servizi o [Services](https://kubernetes.io/docs/concepts/services-networking/service/).

```shell
k expose deployment nginx --port 80
```

Grazie all’addon DNS che abbiamo attivato in precedenza, `nginx` è un nome risolvibile da qualunque Pod nel nostro cluster. Quindi proveremo a contattare Nginx dal nostro BusyBox `pod2`.

```shell
k exec pod2 -- wget -O- nginx
```

![Kubernetes Service Running](/assets/img/2022/12/service-running.jpg)

## Ingress

![Kubernetes Ingress](/assets/img/2022/12/ingress.jpg)

L’ultimo passo di questo tutorial è esporre il nostro web server al di fuori del nostro Kubernetes cluster. In questo caso, useremo il concetto di [Ingress](https://kubernetes.io/docs/concepts/services-networking/ingress/).

```shell
k create ingress nginx --class nginx --rule="/=nginx:80"
```

Adesso puoi raggiungere la pagina di benvenuto anche al di fuori del cluster.

![Nginx Welcome Page](/assets/img/2022/12/nginx-welcome.jpg)

Se dovessi avere problemi prova a cambiare il parametro `--class nginx` con `--class public`.

## Conclusione

In pochi minuti abbiamo installato Kubernetes e imparato a gestire i costrutti principali. Se vuoi approfondire fai riferimento alla documentazione ufficiale o lasciami un messaggio.

Alla prossima!