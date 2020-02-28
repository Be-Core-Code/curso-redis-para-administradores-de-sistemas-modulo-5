### Sentinel

Demonio que monitoriza las instancias de Redis y reconfigura las instancias (maestro y réplicas) en función del
estado de los nodos.

Para garantizar la alta disponibilidad del cluster es necesario que existan varias instancias del proceso Sentinel.

^^^^^^

#### Sentinel: Responsabilidades

* Monitorización: verifica que el maestro y las réplicas están funcionando correctamente
* Notificación: nos avisa si alguno de los nodos se ha caído

^^^^^^

#### Sentinel: Responsabilidades

* Failover:  Si el maestro no está funcionando correctamente, Sentinel inicia un proceso de failover para 
  convertir una de las réplicas en maestro y configurar el resto de réplicas para que utilicen el nuevo maestro.
  También permite avisar a los clientes (nuestra app por ejemplo) de que hay un nuevo maestro.
* Gestor de configuración: actúa como un "Source of Authority" y permite a los clientes conocder cuál es la dirección del maestro. 
  También es capaz de notificar a los clientes cuando este ha cambiado.
^^^^^^

#### Sentinel

La decisión de failover del maestro utiliza un sistema de `quorum` por lo que 
al menos se requiren tres procesos de Sentinel que monitoricen el estado del nodo maestro.
 
^^^^^^

#### Sentinel
 
Con la configuración adecuada, este proceso se realiza de manera automática y no requiere interveción por nuestra parte 
(salvo para levantar el nodo que se ha caído)

^^^^^^

#### Sentinel

Sentinel está diseñado también como un sistema distribuido. Es decir, tendremos varios procesos Sentinel trabajando
de forma coordinada para mantener nuestro sistema en buena salud.

^^^^^^

#### Sentinel

* Para determinar que un nodo está caído, los procesos Sentinel que estar de acuerdo en que lo está. 
  Esto permite reducir el número de falsos positivos.

* Sentinel el robusto: el propio Sentinel pueden funcionar si alguno de los procesos deja de funcionar

^^^^^^

#### Sentinel

Si váis a desplegar clusters de Redis **leed con detenimiendo 
[la documentación completa](https://redis.io/topics/sentinel) al respecto**

notes:

Debido a la limitación de tiempo de los cursos, no podemos entrar en el detalle completo de configuración y funcionamiento
de Sentinel. Como os indico en la diapositiva, por favor leed esta sección de la documentación con atención
si váis a montar un cluster de Redis en producción. Si no entendéis bien el funcionamiento de Sentinel, podéis
perder datos en caso de caída de los nodos o fallos de red. 