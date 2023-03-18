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

¡Hola! En los últimos meses, gracias en parte a mi trabajo, he estado inmerso en el aprendizaje de un fascinante mundo de sistemas distribuidos, sistemas de actores, CRDTs y demás. Es sorprendente cómo hay una escasez de material de fácil digestión cuando buscas información concreta sobre estos temas. La mayoría de las veces, me encuentro que hay que leer artículos o publicaciones de investigación muy técnicos y detallados para poder entender un tema. El problema se acentúa aún más si se busca información en español. En español directamente no hay casi contenido. 


Es por eso que he decidido comenzar una serie de publicaciones en mi blog, en la que explicaré los conceptos básicos de los sistemas distribuidos, construyendo desde cero un key-value store distribuido. 
Esta vez he decido hacerlo en español ya que no he encontrado muchas resources buenas del tema en el idioma. Espero que sea de utilidad a cualquier persona que quiera aprender sobre distributed systems con un pace mucho mas ameno y driendly que esos scary-looking buzz-sounding papers que se publican de vez en cuando. Creo que será una forma práctica y amena de acercarnos a estos temas, y espero que te animes a seguirla.


La idea de esto no es producir para nada algo que sea *production ready*. Sino que lo voy a usar como una excusa para hacer mi primer proyecto real con Go (he realizado el Tour of Go y he hecho algunas PR en varops repositorios del trabajo ya que Datadog usa bastante Go) y entender de verdad algunas de las buzzwords que suelo oir en todos los posts de high scalability como Raft, rebalancing, Gorillas compaction o CRDTs. Como dijo Feynman: Si no lo puedo crear, no lo entiendo. 


Basicamente sera un proyecto tipo Mr.Potato al que le pondre cuantas mas cosas pueda mejor siempre y que tenga sentido. Intentare seguir las mejores practicas y documentarlo todo en mi blog. 


## Plan inicial



