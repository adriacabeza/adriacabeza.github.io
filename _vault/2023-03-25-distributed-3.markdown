---
layout: post
title: "Crear un una base de datos distribuida from scratch III: RAFT"
date: 2023-02-18
tags:
  - Sistemas distribuidos
  - Storage
  - Raft
  - Golang
---
# Capitulo 3: Raft
Se viene lo interesante. 

https://raft.github.io/raft.pdf


El algoritmo Raft es un algoritmo de consenso distribuido que se utiliza para mantener la coherencia de un grupo de nodos en un sistema distribuido. El objetivo del algoritmo es permitir que los nodos del sistema lleguen a un acuerdo sobre un valor común, incluso si algunos nodos fallan o pierden la conexión con el resto del sistema. Nuestro definicion de nodo en el codigo sera la siguiente:
```
type Node struct {
	id        int
	term      int
	votedFor  int
	state     string
	leaderId  int
	timeout   *time.Timer
	voteCount int
	votes     map[int]bool
}
```

El algoritmo Raft está diseñado para ser fácil de entender. Anterior a el, estaba Paxos, desarollado por el gran Leslie Lamport. Pero desafortunadamente, Paxos tiene varios problemas significativos. El primero de ellos, y como ya mencionamos en la introducción, es la complejidad para ser entendido debido a que su explicación es larga y opaca. Poca gente logra tener éxito en su comprensión y para los afortunados que lo consiguen, ha sido a través de un grandísimo esfuerzo.  

 Raft se compone de tres subproblemas principales: elección de líder, replicación de registros y seguridad. Cada uno de estos subproblemas se aborda mediante una serie de reglas y protocolos que gobiernan el comportamiento de los nodos del sistema. 

 Elección de líder: En un sistema distribuido, es necesario que un nodo actúe como líder para coordinar las operaciones de los demás nodos. Sin embargo, en caso de que el líder falle o pierda la conexión, es necesario elegir un nuevo líder. El algoritmo Raft utiliza un proceso de elección de líder que se basa en el concepto de términos. Cada término tiene un líder que es responsable de coordinar las operaciones de los nodos durante ese período de tiempo. Cuando un nodo se inicia por primera vez, comienza un nuevo término y se convierte en candidato para liderar el sistema. El candidato envía solicitudes de voto a los demás nodos, y si recibe la mayoría de los votos, se convierte en líder del término. Si un nodo no recibe suficientes votos, se inicia un nuevo término y se repite el proceso.

Replicación de registros: Una vez que se ha elegido un líder, es necesario garantizar que los registros del sistema se repliquen en todos los nodos. El líder es responsable de recibir las solicitudes de escritura de los clientes, replicar los registros en los nodos seguidores y confirmar la escritura una vez que se han replicado en la mayoría de los nodos seguidores. Los seguidores están a la espera de recibir solicitudes de escritura del líder y replicar los registros. Si un seguidor no recibe registros del líder durante un tiempo determinado, se convierte en candidato y comienza el proceso de elección de líder.

Seguridad: Finalmente, es necesario garantizar que el sistema sea seguro y resistente a los fallos. El algoritmo Raft utiliza una serie de mecanismos para garantizar la seguridad del sistema, incluyendo la votación por mayoría, la replicación de registros y la notificación de fallos. Si un nodo sospecha que otro nodo ha fallado o ha perdido la conexión, notifica al resto de los nodos y se inicia el proceso de elección de líder para elegir un nuevo líder.




 que un cluster de nodos puede mantener una state machine replicada. La state machine se mantiene en sync mediante un repliacted log. Dentro del log se almacenan una serie de comandos que la maquina de estados debe ser ejecutada en perfecto orden. Todos los logs de los diferentes serviceores deben almacenar los mismos comandos en las mismas posiciones para que pueda considerarse su ejecucion determinista. 







Raft fue diseñado para ser comprensible. 

El lider es elegidoentre los diferentes servideores a traves de votaciones. Cuando un servidor consigue la mayoria de los votos es proclamado lider. La principal responsabilidad del lider es gestionar el log replicado. Para ello el lider es le unico que puede aceptar comandos de clientes, replicar estos comandos en los diferentes servidores y comunicarles cuando es seguro ejecutarlo en sus maquinas de estados. 

Si el lider falla o se deconecta de los demas servidores, el proceso de eleccion de un nuevo lider volvera a realizarse. 



Propiedades que Raft asegura:
1. Election safety: solo un lider en una legislatura
2. Solo los lideres añaden al log y es lo unico que pueden hacer (ni borrar ni sobreescribir) (Leader Append-Only)
3. State machine safety: si un servidor aplica un comando a la state machine con un indice concreto. Los demas servidores no podran aplicar un comando diferente en el mismo indice. 


Tenemos a tres estados para cada nodo:
 1. Follower: cuando un servidor arranca siempre lo hace en este estado. Son pasivos, simplemente responded a calls del lider y candidatos. No responden a peticiones de los clientes, solo informatn al cliente de quien es el lider. 
 2. Lider: En funcionamento normal, siempre habra un lider, y los demas servidores seran seguidores. Los lideres se encargan de gestionar las peticiones del cliente. 
 3. Candidato: Son usados para la elecciond el nuevo lider.


 Raft divide el tiempo en terms (legislaturas) numerados consecutivamente. Cada legislatura comienza con la eleccion de un lider. Si un candidato consigue ser lider, conservara el liderazgo por el resto del term. Si durante las elecciones, ningun servidor consigue ser lider, ese term acaba sin lider y se empieza uno nuevo. Raft tiene que asegurar que durante un term no puede haber mas de un lider. 

 Uno de los problemas que surge es la posibilidad de que varios servidores tengag diferentes terms debido a problemas de connexion o aislamiento entre servidores. 

 1. Si un servidor esta en un term mas pequeno que otro entonces esta obligado a actualizar su term
 2. Si un servidor que esta en estado de lider o candidato descubre que esta desactualizado, tiene que pasar su estado a seguidor
 3. Si un servidor recibe una solicitud de otro servidor con una legislatura mas antigua que la actual, se rechaza la peticion. 

```
type Message struct {
	from        int
	to          int
	term        int
	vote        bool
	appendEntry bool
}
```

AppendEntries tiene dos modalidades:
- Hearbeat: sirve para comunicar al resto de servidores que el lider esta vido
- Replicar logs: su mision es enviar el resto de los servidores los nuevos comandos que tienen que guardar en su log y posteriormente, si son aceptados por la mayoria, aplicar los en su maquina de estados. 


Capitulo: Observability 
- Pprof: https://hmarr.com/blog/go-allocation-hunting/
- Add Logging 
- Add Jaeger

Para el capitulo de improvements: Performance
- Aprender como hacer profiling 
- Cambiar net/http por fasthttp
- Mirar trucos per millorar Golang Performance  


