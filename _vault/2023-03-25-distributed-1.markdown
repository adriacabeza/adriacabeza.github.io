---
layout: post
title: "Crear un una base de datos distribuida from scratch I: planificacion"
date: 2023-02-18
tags:
  - Sistemas distribuidos
  - Storage
  - Raft
  - Golang
---

👋 Saludos developer intrepido/a! 

 En los últimos meses, gracias en parte a mi trabajo en Datadog, he estado inmerso en el aprendizaje de un fascinante mundo de sistemas distribuidos, sistemas de actores, CRDTs y demás. 
 
Es sorprendente cómo hay una escasez de material de fácil digestión cuando buscas información concreta sobre estos temas. La mayoría de las veces, me encuentro que hay que leer artículos o publicaciones de investigación muy técnicos y detallados para poder entender un tema. El problema se acentúa aún más si se busca información en español. En español directamente no hay casi contenido. 


Es por eso que he decidido comenzar una serie de publicaciones en mi blog, en la que explicaré los conceptos básicos de los sistemas distribuidos, construyendo desde cero un key-value store distribuido. Espero que sea de utilidad a cualquier persona que quiera aprender sobre distributed systems con un ritmo mucho mas ameno y mas friendly. 

La idea de esto no es producir para nada algo que sea *production ready*. Sino que lo voy a usar como una excusa para entender de verdad algunas de las buzzwords que suelo oir en todos los posts de high scalability como Raft, Paxos, Gorillas compaction o CRDTs. Como dijo Feynman: Si no lo puedo crear, no lo entiendo. 


## Plan inicial



