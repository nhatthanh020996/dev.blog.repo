---
layout:     post
title:      "Isolation"
description: "Isolation là tính chất bị underated nhất trong ACID, trong bài viết này
mình xin trình bày rõ hơn về tính chất này cũng như đưa ra các trường hợp cần setup đúng
isolation level trong thực tế"
date:    2021-11-04T12:00:00
author:     "Admin"
image: "/img/posts/isolation/title_image.jpeg"
tags:
    - ACID
    - Database
    - PostgreSQL
URL: "/testpage/"
categories: [ Tech ]
---

## 服务网格简介

**服务网格**（Service Mesh）是为解决微服务的通信和治理而出现的一种**架构模式**。

服务网格将服务间通讯以及与此相关的管理控制功能从业务程序中下移到一个基础设施层，从而彻底隔离了业务逻辑和服务通讯两个关注点。采用服务网格后，应用开发者只需要关注并实现应用业务逻辑。服务之间的通信，包括服务发现，通讯的可靠性，通讯的安全性，服务路由等由服务网格层进行处理，并对应用程序透明.

<!--more-->
让我们来回顾一下微服务架构的发展过程。在出现服务网格之前，我们在微服务应用程序进程内处理服务通讯逻辑，包括服务发现，熔断，重试，超时等逻辑，如下图所示：  
![](/img/istio-install_and_example/5-a.png)  
通过对这部分负责服务通讯的逻辑进行抽象和归纳，可以形成一个代码库供应用程序调用。但应用程序还是需要处理和各种语言代码库的调用细节，并且各种代码库互不兼容，导致对应用程序使用的语言和代码框架有较大限制。

如果我们更进一步，将这部分逻辑从应用程序进程中抽取出来，作为一个单独的进程进行部署，并将其作为服务间的通信代理，如下图所示：  
![](/img/istio-install_and_example/6-a.png)  
因为通讯代理进程和应用进程一起部署，因此形象地把这种部署方式称为“sidecar”（三轮摩托的挎斗）。
![](/img/istio-install_and_example/sidecar.jpg)
应用间的所有流量都需要经过代理，由于代理以sidecar方式和应用部署在同一台主机上，应用和代理之间的通讯被认为是可靠的。然后由代理来负责找到目的服务并负责通讯的可靠性和安全等问题。

当服务大量部署时，随着服务部署的sidecar代理之间的连接形成了一个如下图所示的网格，被称之为Service Mesh（服务网格），从而得出如下的服务网格定义。

_服务网格是一个基础设施层，用于处理服务间通信。云原生应用有着复杂的服务拓扑，服务网格保证请求可以在这些拓扑中可靠地穿梭。在实际应用当中，服务网格通常是由一系列轻量级的网络代理组成的，它们与应用程序部署在一起，但应用程序不需要知道它们的存在。_

_William Morgan _[_WHAT’S A SERVICE MESH? AND WHY DO I NEED ONE?_](https://buoyant.io/2017/04/25/whats-a-service-mesh-and-why-do-i-need-one/)_                                               _

![](/img/istio-install_and_example/mesh1.png)