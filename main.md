---
abstract: 'TO--DO'
author:
- Javier Alonso Silva
bibliography:
- 'base/Docker.bib'
date: 2021
title: Análisis de contenedores Docker
---

![image](upmlogo.png){width="\linewidth"}

![image](dockerlogo.png){width="50%"}

\- y sus implicaciones de seguridad\

------------------------------------------------------------------------

\

Seguridad en Sistemas y Redes

Universidad Politécnica de Madrid

Introducción
============

La era tecnológica ha avanzado en los últimos años a pasos agigantados,
y las demandas del sector han crecido junto a ella. No hace más de 200
años se "descubría" la electricidad; hace 90 años nacía la primera
computadora básica capaz de realizar operaciones aritméticas; hace 70
años nacía el transistor que sustituyó las válvulas de vacío (figura
[1](#fig:transistor){reference-type="ref" reference="fig:transistor"});
y desde entonces, el crecimiento ha sido exponencial
[@HistoryTechnologyTimeline].

![Comparativa de una válvula de vacío (izquierda) frente a un transistor
(centro) y un circuito integrado
(derecha).[]{label="fig:transistor"}](pictures/transistor-vs-valve.jpg){#fig:transistor
width=".7\linewidth"}

Otro de los ejemplos de tecnologías que han crecido exponencialmente son
los dispositivos de almacenamiento, donde no hacía más de 20 años las
capacidades máximas se estimaban en torno a los MB (megabytes) y ahora
se hablan de EB (exabytes) [@EvolutionDataStorage]. Esta evolución es
muy representativa también a nivel económico, ya que el coste del
almacenamiento ha ido bajando a medida pasaba el tiempo, así como el
espacio físico que ocupan los dispositivos (figura
[2](#fig:disk-evolution){reference-type="ref"
reference="fig:disk-evolution"}):

![Evolución del espacio de almacenamiento en términos económicos y
cuantitativos
[@wecomputingtechStorageDevicesLondon].[]{label="fig:disk-evolution"}](pictures/disk-evo.jpg){#fig:disk-evolution
width=".65\linewidth"}

Finalmente, el gran salto tecnológico se ha producido con la aparición
de Internet y las comunicaciones ya no eran únicamente personales sino
entre dispositivos. En relación con el punto anterior, la aparición de
Internet ha permitido descentralizar el espacio donde ya el usuario no
guarda su información en su equipo personal sino en un clúster de
servidores distribuidos a nivel mundial al cual accede, de forma
simultánea, desde Internet y desde cualquier dispositivo. Así, lo que
comenzó como una red de conexión de unos pocos usuarios ha acabado
convirtiéndose en la red global que todos usamos y que conecta más de 4
billones de dispositivos (figura
[3](#fig:internet-evo){reference-type="ref"
reference="fig:internet-evo"}).

![Evolución de Internet a lo largo del tiempo, hasta llegar a hoy
[@HowBecomeWeb].[]{label="fig:internet-evo"}](pictures/internet-timeline.png){#fig:internet-evo
width=".9\linewidth"}

El problema a esto es evidente: con una mayor capacidad de cómputo, con
más opciones de comunicación y con más posibilidad de almacenar datos,
los requisitos de las aplicaciones van creciendo y creciendo y cada vez
son más complejos de satisfacer, no necesariamente a nivel *hardware*
(que por lo general suele acompañar) sino a nivel *software*. Como las
aplicaciones se orientan a los usuarios es necesario añadir capas de
abstracción (como el sistema operativo) para facilitar la labor a la
persona. Sin embargo, cada capa nueva que se añade dificulta las tareas
de despliegue y mantenimiento dado que existe una gran variedad de
combinaciones *hardware* y cada una puede estar con un sistema operativo
distinto.

Por otra parte, la extensión de dependencias y posible incompatibilidad
entre ellas suele desembocar en el uso de versiones desactualizadas de
una librería ya que tendríamos "paquetes rotos". Esto es tan común que
tiene hasta su propio término coloquial "*dependency hell*"
[@DependencyHell2021]. Contar con dependencias obsoletas que ya han
cumplido con su ciclo de vida *software* conlleva unas implicaciones de
seguridad bastante severas:

-   Si un *software* no ha mejorado a lo largo del tiempo, existe una
    malicia humana que puede aprovecharse de distintos *exploits*
    existentes y comprometan nuestra aplicación.

-   Un *software* no actualizado puede tener implicaciones directas
    sobre el sistema en que se ejecuta, pudiendo producir fallos en el
    mismo. Esto se debe principalmente a que el *hardware* sigue
    mejorando y creciendo y un *software* antiguo puede presentar *bugs*
    en dispositivos modernos que no presentaría en antiguos.

-   Un *software* no actualizado puede comprometer otros elementos del
    sistema en que se ejecuta. Por ejemplo, una aplicación 'A' hace uso
    de dicho *software* y una aplicación 'B' también. Sin embargo, la
    última aplicación se ha diseñado para trabajar con la última versión
    del *software* pero la aplicación 'A' solo puede funcionar con una
    versión antigua e insegura. Por consiguiente, pese a que la
    aplicación 'B' funcionaría correctamente el hecho de usar una
    versión antigua e insegura del *software* compromete directamente al
    sistema y a la aplicación.

Es por eso que existen alternativas como "`chroot`" y máquinas virtuales
para subsanar estos problemas. Sin embargo, en los últimos años ha
aparecido una herramienta muy sonada y con gran éxito: Docker y los
contenedores.

¿Qué es Docker?
---------------

Docker es una plataforma abierta diseñada para el desarrollo, despliegue
y ejecución de aplicaciones [@DockerOverview2021]. La idea fundamental
que reside detrás de Docker es la de separar la infraestructura de las
aplicaciones de manera que se pueda entregar el *software* rápidamente.

Por debajo, Docker ofrece una plataforma que otorga la habilidad de
empaquetar y ejecutar las aplicaciones en un entorno aislado llamado
"contenedor" (*container*). Entre otras características, un contenedor
permite ejecutar una aplicación de forma segura sobre el host en
cuestión. La pregunta que surge es, ¿qué es un contenedor?

### Contenedores {#contenedores .unnumbered}

Un contenedor es una unidad estándar *software* que empaqueta código y
todas sus dependencias de manera que la aplicación se ejecuta
rápidamente y de forma fiable bajo múltiples entornos de ejecución
[@WhatContainerApp]. Una imagen Docker es un paquete ligero,
independiente y ejecutable que incluye absolutamente todo lo necesario
para poder ejecutar una aplicación: desde el código en sí hasta el
*runtime*, herramientas del sistema, librerías y configuraciones.

Durante la ejecución, una imagen se convierte en un contenedor que se
ejecuta sobre la máquina Docker (*Docker Engine*), la cual se encuentra
disponible en entornos Linux y Windows.

Al fin y al cabo, los contenedores nos aseguran que una aplicación que
hemos desarrollado se va a ejecutar de la misma manera en una máquina u
otra. El uso del motor Docker permite ejecutar múltiples contenedores
sobre un mismo anfitrión sin añadir demasiada carga en el sistema e
indiferentemente de la infraestructura que exista por debajo (figura
[4](#fig:docker-containers-rt){reference-type="ref"
reference="fig:docker-containers-rt"}):

![Distribución de los contenedores sobre el motor de ejecución de Docker
[@WhatContainerApp].[]{label="fig:docker-containers-rt"}](pictures/container-what-is-container.png){#fig:docker-containers-rt
width=".7\linewidth"}

La distribución de los contenedores mostrada en la figura
[4](#fig:docker-containers-rt){reference-type="ref"
reference="fig:docker-containers-rt"} puede parecerse mucho a la
distribución que tendríamos en una máquina virtual. Sin embargo, hay
varias características que lo distinguen principalmente:

1.  Un contenedor se ejecuta directamente sobre la máquina anfitriona,
    mientras que una máquina virtual requiere de un hipervisor.

2.  Un contenedor es una abstracción de la capa de aplicación que
    encapsula el código y las dependencias juntas, mientras que una
    máquina virtual es una abstracción de una capa física *hardware*.

3.  Un contenedor comparte el kernel con el sistema operativo anfitrión,
    por lo que tiene un gran rendimiento; mientras, una máquina virtual
    ejecutará su propio kernel sobre el hipervisor del sistema operativo
    anfitrión.

4.  El espacio que necesita un contenedor es muy pequeño en comparación
    con el de una máquina virtual, que engloba y encapsula un sistema
    operativo al completo.

![Capas de abstracción de una máquina virtual sobre una máquina
anfitriona
[@WhatContainerApp].[]{label="fig:vm-layers"}](pictures/container-vm-whatcontainer_2.png){#fig:vm-layers
width=".7\linewidth"}

En la figura [5](#fig:vm-layers){reference-type="ref"
reference="fig:vm-layers"} se puede apreciar cómo una máquina virtual
añade muchas más capas de abstracción que ralentizan el rendimiento. Sin
embargo, esto no quiere decir que sean una mala alternativa: la realidad
es que se combinan las dos para obtener una gran flexibilidad para
desplegar aplicaciones -- contenedores cuando se quiere ejecutar algo
directamente sobre el anfitrión; máquinas virtuales para emular
*hardware* y que ejecuten en su interior contenedores para ejecutar
aplicaciones fácilmente.

La evolución y constante mantenimiento de los contenedores ha generado
lo que se conoce como estándar de la industria "*containerd*". Este
estándar define claramente qué arquitectura debe tener un contenedor por
debajo y está en constante evolución a medida que la industria crece y
madura.

Además, la especificación anterior ha pasado de ser un mero estándar a
una aplicación en sí de gestión y orquestación de contenedores,
permitiendo que aplicaciones distintas de Docker hagan uso de la
arquitectura basada en contenedores aprovechando la OCI: *Open Container
Initiative*

![Entorno de ejecución de *containerd* basado en `runC` de la OCI
[@Containerd].[]{label="fig:containerd-arch"}](pictures/containerd-arch.png){#fig:containerd-arch
width=".7\linewidth"}

### Docker Engine {#docker-engine .unnumbered}

El motor de ejecución de Docker establece la arquitectura de ejecución
*de facto* que es utilizable desde distintas distribuciones Linux y
servidores Windows [@ContainerRuntimeDocker].

![Arquitectura del entorno de ejecución de Docker
[@ContainerRuntimeDocker].[]{label="fig:docker-engine"}](pictures/docker-engine.png){#fig:docker-engine
width=".7\linewidth"}

El motor de ejecución de Docker se compone de una gran cantidad de
elementos que encapsulan de forma uniforme multitud de aptitudes de un
sistema operativo o una aplicación (figura
[7](#fig:docker-engine){reference-type="ref"
reference="fig:docker-engine"}). Este entorno de ejecución sin embargo
es complejo ya que engloba multitud de elementos físicos, como pueden
ser las interfaces de red y los volúmenes.

Esto resulta fundamental ya que los contenedores Docker no tienen ni que
confiar en la red del anfitrión: tienen su propio *stack* de red para
realizar las comunicaciones que necesiten. Con el motor de ejecución de
Docker se busca solventar esos problemas "*dependency hell*" que se han
comentado anteriormente y la situación de "en mi equipo funciona".

De los elementos mostrados en la figura
[7](#fig:docker-engine){reference-type="ref"
reference="fig:docker-engine"}, se tiene que son:

-   *Distribution*: la distribución Linux en la que se basa el
    contenedor. Actualmente, Docker solo permite ejecutar contenedores
    basados en Linux.

-   *Orchestration*: cuando hay múltiples contenedores, la orquestación
    es el proceso por el cual el motor de ejecución de Docker gestiona y
    maneja qué contenedores se ejecutan, cómo se comunican, cuáles hay
    que crear nuevos y cuáles eliminar. Es de las partes más complejas
    que existen en el mundo de los contenedores y ha evolucionado a
    clústers mucho más completos (y complejos) como Kubernetes o Docker
    Swarm.

-   *Volumes*: los volúmenes (conjuntos de datos) que se manejan en los
    contenedores. Debido a su arquitectura cerrada, los datos que genera
    un contenedor solo están visibles para ese contenedor mientras este
    esté en ejecución. Cuando finalice, todos los datos no persistentes
    son eliminados.

-   *Containerd*: el estándar y cliente de ejecución y manejo de los
    contenedores a muy bajo nivel.

-   *Docker Build (BuildKit)*: herramienta de libre distribución que
    transforma los ficheros `Dockerfile` en imágenes Docker, listas para
    ser usadas y distribuídas.

-   *Networking*: *stack* de red completo que se pone a disposición de
    cada contenedor Docker. Cada aplicación puede crear su propio
    dispositivo de red que cumpla con los requisitos que necesita.
    Existen varios tipos de adaptadores: *bridge*, *NAT* y *host*. El
    primero se emplea para realizar comunicaciones a través de red entre
    distintos contenedores; el segundo para realizar comunicaciones con
    el exterior mediante una conexión de red completamente independiente
    a la del anfitrión; la tercera para compartir la interfaz de red del
    anfitrión con el contenedor, como si fuese una aplicación interna.

Con todo lo anterior, una aplicación puede ejecutarse muy fácilmente en
cualquier equipo que integre el motor de ejecución de Docker.

------------------------------------------------------------------------

Con esto, ya sabemos

*Real--life usages*
-------------------

*Docker rules*
--------------

Docker
======

Estructura de un Docker
-----------------------

Creación de un contenedor
-------------------------

Comunicación entre contenedores
-------------------------------

Despliegue de aplicaciones multi--contenedores. `docker-compose`
----------------------------------------------------------------

"Orquestación" de contenedores
------------------------------

Líneas futuras de desarrollo e innovación
-----------------------------------------

Seguridad en Docker
===================

Análisis de la pila Docker
--------------------------

Diferencias fundamentales con `chroot`
--------------------------------------

Seguridad en las comunicaciones de red -- *firewall*
----------------------------------------------------

Seguridad en las comunicaciones inter--contenedores
---------------------------------------------------
