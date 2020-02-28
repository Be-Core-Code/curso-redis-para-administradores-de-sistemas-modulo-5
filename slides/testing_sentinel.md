### Probando la configuración

Vamos a probar la configuración de varias maneras:

* Caso 1: Activando un failover maualmente
* Caso 2: Apagando el proceso Redis en el maestro
* Caso 3: Tiramos Redis en las dos réplicas
* Caso 4: Tiramos un proceso Sentinel
* Caso 5: Tiramos dos procesos Sentinel 

^^^^^^

#### 💻️ Probando la configuración: Caso 1

* Nos conectamos al proceso Sentinal de la `replica1`:

```bash
(replica1) > redis-cli -h 192.168.155.147 -p 26379
```


^^^^^^

#### 💻️ Probando la configuración: Caso 1

* Ejecutamos el comando `SENTINEL FAILOVER`

```redis-cli
redis-cli (replica1) > SENTINEL FAILOVER cursoredis
OK
```

* Estamos pidiendo que se lleve a cabo un failover.
* Los procesos Sentinel dedidirán quién es el nuevo maestro
  y adaptarán la configuración de Redis
    
^^^^^^

#### 💻️ Probando la configuración: Caso 1

* Vamos a mirar el log de Sentinel en la `replica1` (donde lanzamos el comando `SENTINEL FAILOVER`) :

```
4869:X 28 Feb 2020 14:48:01.682 # Executing user requested FAILOVER of 'cursoredis'
4869:X 28 Feb 2020 14:48:01.682 # +new-epoch 3
4869:X 28 Feb 2020 14:48:01.682 # +try-failover master cursoredis 192.168.157.144 6379
4869:X 28 Feb 2020 14:48:01.683 # +vote-for-leader c7e03c1ddaca2b28a496a6e585c02340ed1e25ff 3
4869:X 28 Feb 2020 14:48:01.683 # +elected-leader master cursoredis 192.168.157.144 6379
4869:X 28 Feb 2020 14:48:01.683 # +failover-state-select-slave master cursoredis 192.168.157.144 6379
4869:X 28 Feb 2020 14:48:01.741 # +selected-slave slave 192.168.157.148:6379 192.168.157.148 6379 @ cursoredis 192.168.157.144 6379
4869:X 28 Feb 2020 14:48:01.741 * +failover-state-send-slaveof-noone slave 192.168.157.148:6379 192.168.157.148 6379 @ cursoredis 192.168.157.144 6379
4869:X 28 Feb 2020 14:48:01.818 * +failover-state-wait-promotion slave 192.168.157.148:6379 192.168.157.148 6379 @ cursoredis 192.168.157.144 6379
4869:X 28 Feb 2020 14:48:02.745 # +promoted-slave slave 192.168.157.148:6379 192.168.157.148 6379 @ cursoredis 192.168.157.144 6379
4869:X 28 Feb 2020 14:48:02.745 # +failover-state-reconf-slaves master cursoredis 192.168.157.144 6379
4869:X 28 Feb 2020 14:48:02.817 * +slave-reconf-sent slave 192.168.157.147:6379 192.168.157.147 6379 @ cursoredis 192.168.157.144 6379
4869:X 28 Feb 2020 14:48:03.757 * +slave-reconf-inprog slave 192.168.157.147:6379 192.168.157.147 6379 @ cursoredis 192.168.157.144 6379
4869:X 28 Feb 2020 14:48:03.757 * +slave-reconf-done slave 192.168.157.147:6379 192.168.157.147 6379 @ cursoredis 192.168.157.144 6379
4869:X 28 Feb 2020 14:48:03.846 # +failover-end master cursoredis 192.168.157.144 6379
4869:X 28 Feb 2020 14:48:03.847 # +switch-master cursoredis 192.168.157.144 6379 192.168.157.148 6379
4869:X 28 Feb 2020 14:48:03.847 * +slave slave 192.168.157.147:6379 192.168.157.147 6379 @ cursoredis 192.168.157.148 6379
4869:X 28 Feb 2020 14:48:03.847 * +slave slave 192.168.157.144:6379 192.168.157.144 6379 @ cursoredis 192.168.157.148 6379
```

^^^^^^

#### 💻️ Probando la configuración: Caso 1

* Vamos a mirar el log de Sentinel en la `replica2` (que ha sido seleccionado como nuevo master):

```
4953:X 28 Feb 2020 14:48:00.817 # +new-epoch 3
4953:X 28 Feb 2020 14:48:00.817 # +config-update-from sentinel c7e03c1ddaca2b28a496a6e585c02340ed1e25ff 192.168.157.147 26379 @ cursoredis 192.168.157.144 6379
4953:X 28 Feb 2020 14:48:00.817 # +switch-master cursoredis 192.168.157.144 6379 192.168.157.148 6379
4953:X 28 Feb 2020 14:48:00.818 * +slave slave 192.168.157.147:6379 192.168.157.147 6379 @ cursoredis 192.168.157.148 6379
4953:X 28 Feb 2020 14:48:00.818 * +slave slave 192.168.157.144:6379 192.168.157.144 6379 @ cursoredis 192.168.157.148 6379
4953:X 28 Feb 2020 14:48:10.867 * +convert-to-slave slave 192.168.157.144:6379 192.168.157.144 6379 @ cursoredis 192.168.157.148 6379
```

^^^^^^

#### 💻️ Probando la configuración: Caso 1

* Vamos a mirar el log de Sentinel en el antiguo `maestro`:

```
5248:X 28 Feb 2020 14:48:03.962 # +new-epoch 3
5248:X 28 Feb 2020 14:48:03.962 # +config-update-from sentinel c7e03c1ddaca2b28a496a6e585c02340ed1e25ff 192.168.157.147 26379 @ cursoredis 192.168.157.144 6379
5248:X 28 Feb 2020 14:48:03.962 # +switch-master cursoredis 192.168.157.144 6379 192.168.157.148 6379
5248:X 28 Feb 2020 14:48:03.963 * +slave slave 192.168.157.147:6379 192.168.157.147 6379 @ cursoredis 192.168.157.148 6379
5248:X 28 Feb 2020 14:48:03.963 * +slave slave 192.168.157.144:6379 192.168.157.144 6379 @ cursoredis 192.168.157.148 6379
```

^^^^^^

#### 💻️ Probando la configuración: Caso 1

* ¿Cómo ha quedado configuración de los procesos Sentinel?

```
(maestro) > grep sentinel\ monitor /etc/sentinel.conf
sentinel monitor cursoredis 192.168.157.148 6379 2
```

notes: 

Se ha actualizado la configuración de Sentinel en todos los nodos para indicar quién es el nuevo maestro.

^^^^^^

#### 💻️ Probando la configuración: Caso 1

```
(replica1) > grep sentinel\ monitor /etc/sentinel.conf
sentinel monitor cursoredis 192.168.157.148 6379 2
```

```
(replica2) > grep sentinel\ monitor /etc/sentinel.conf
sentinel monitor cursoredis 192.168.157.148 6379 2
```

^^^^^^

#### 💻️ Probando la configuración: Caso 1

* ¿Cómo ha quedado configuración de Redis?

```redis-cli
redis-cli (maestro) > INFO Replication
# Replication
role:slave                             <---- Ha dejado de ser
master_host:192.168.157.148            <---- el maestro del cluster
master_port:6379
master_link_status:up
master_last_io_seconds_ago:0
... 
```

^^^^^^

#### 💻️ Probando la configuración: Caso 1

```redis-cli
redis-cli (replica2) > INFO Replication
# Replication
role:master                  <---- Esta réplica es el nuevo maestro
connected_slaves:2           <---- 
slave0:ip=192.168.157.147,port=6379,state=online,offset=1873677,lag=0
slave1:ip=192.168.157.144,port=6379,state=online,offset=1873677,lag=0
... 
```

^^^^^^

#### Probando la configuración

Vamos a probar la configuración de dos maneras:

* ✅ Caso 1: Activando un failover manualmente
* Caso 2: Apagando el proceso Redis en el maestro
* Caso 3: Tiramos Redis en las dos réplicas
* Caso 4: Tiramos un proceso Sentinel
* Caso 5: Tiramos dos procesos Sentinel 

^^^^^^

#### 💻️ Probando la configuración: Caso 2

* Continuamos donde lo dejamos en el paso anterior: el nodo `replica2` es
  el nuevo maestro
* En esta situación, tiramos el nuevo maestro (`replica2`)

```redis-cli
redis-cli (replica2) > SHUTDOWN
not connected> 
```

^^^^^^

#### 💻️ Probando la configuración: Caso 2

* Vamos a mirar el log de Sentinel del nodo `replica1`:

```bash
4869:X 28 Feb 2020 15:22:10.467 # +sdown master cursoredis 192.168.157.148 6379
4869:X 28 Feb 2020 15:22:10.559 # +new-epoch 4
4869:X 28 Feb 2020 15:22:10.560 # +vote-for-leader d7e03c1ddaca2b28a496a6e585c02340ed1e25ff 4
4869:X 28 Feb 2020 15:22:11.390 # +config-update-from sentinel d7e03c1ddaca2b28a496a6e585c02340ed1e25ff 192.168.157.148 26379 @ cursoredis 192.168.157.148 6379
4869:X 28 Feb 2020 15:22:11.390 # +switch-master cursoredis 192.168.157.148 6379 192.168.157.144 6379
4869:X 28 Feb 2020 15:22:11.390 * +slave slave 192.168.157.147:6379 192.168.157.147 6379 @ cursoredis 192.168.157.144 6379
4869:X 28 Feb 2020 15:22:11.390 * +slave slave 192.168.157.148:6379 192.168.157.148 6379 @ cursoredis 192.168.157.144 6379
4869:X 28 Feb 2020 15:22:41.428 # +sdown slave 192.168.157.148:6379 192.168.157.148 6379 @ cursoredis 192.168.157.144 6379
```

^^^^^^

#### 💻️ Probando la configuración: Caso 2

* Vamos a mirar el log de Sentinel del nodo `replica2` (nuevo maestro cuyo proceso redis se ha parado):

```bash
4953:X 28 Feb 2020 15:22:08.466 # +sdown master cursoredis 192.168.157.148 6379
4953:X 28 Feb 2020 15:22:08.552 # +odown master cursoredis 192.168.157.148 6379 #quorum 2/2
4953:X 28 Feb 2020 15:22:08.552 # +new-epoch 4
4953:X 28 Feb 2020 15:22:08.552 # +try-failover master cursoredis 192.168.157.148 6379
4953:X 28 Feb 2020 15:22:08.554 # +vote-for-leader d7e03c1ddaca2b28a496a6e585c02340ed1e25ff 4
4953:X 28 Feb 2020 15:22:08.556 # b7e03c1ddaca2b28a496a6e585c02340ed1e25ff voted for b7e03c1ddaca2b28a496a6e585c02340ed1e25ff 4
4953:X 28 Feb 2020 15:22:08.558 # c7e03c1ddaca2b28a496a6e585c02340ed1e25ff voted for d7e03c1ddaca2b28a496a6e585c02340ed1e25ff 4
4953:X 28 Feb 2020 15:22:08.614 # +elected-leader master cursoredis 192.168.157.148 6379
4953:X 28 Feb 2020 15:22:08.614 # +failover-state-select-slave master cursoredis 192.168.157.148 6379
4953:X 28 Feb 2020 15:22:08.690 # +selected-slave slave 192.168.157.144:6379 192.168.157.144 6379 @ cursoredis 192.168.157.148 6379
4953:X 28 Feb 2020 15:22:08.690 * +failover-state-send-slaveof-noone slave 192.168.157.144:6379 192.168.157.144 6379 @ cursoredis 192.168.157.148 6379
4953:X 28 Feb 2020 15:22:08.763 * +failover-state-wait-promotion slave 192.168.157.144:6379 192.168.157.144 6379 @ cursoredis 192.168.157.148 6379
4953:X 28 Feb 2020 15:22:09.329 # +promoted-slave slave 192.168.157.144:6379 192.168.157.144 6379 @ cursoredis 192.168.157.148 6379
4953:X 28 Feb 2020 15:22:09.329 # +failover-state-reconf-slaves master cursoredis 192.168.157.148 6379
4953:X 28 Feb 2020 15:22:09.385 * +slave-reconf-sent slave 192.168.157.147:6379 192.168.157.147 6379 @ cursoredis 192.168.157.148 6379
4953:X 28 Feb 2020 15:22:09.671 # -odown master cursoredis 192.168.157.148 6379
4953:X 28 Feb 2020 15:22:10.392 * +slave-reconf-inprog slave 192.168.157.147:6379 192.168.157.147 6379 @ cursoredis 192.168.157.148 6379
4953:X 28 Feb 2020 15:22:10.392 * +slave-reconf-done slave 192.168.157.147:6379 192.168.157.147 6379 @ cursoredis 192.168.157.148 6379
4953:X 28 Feb 2020 15:22:10.472 # +failover-end master cursoredis 192.168.157.148 6379
4953:X 28 Feb 2020 15:22:10.472 # +switch-master cursoredis 192.168.157.148 6379 192.168.157.144 6379
4953:X 28 Feb 2020 15:22:10.473 * +slave slave 192.168.157.147:6379 192.168.157.147 6379 @ cursoredis 192.168.157.144 6379
4953:X 28 Feb 2020 15:22:10.473 * +slave slave 192.168.157.148:6379 192.168.157.148 6379 @ cursoredis 192.168.157.144 6379
4953:X 28 Feb 2020 15:22:40.478 # +sdown slave 192.168.157.148:6379 192.168.157.148 6379 @ cursoredis 192.168.157.144 6379
```

^^^^^^

#### 💻️ Probando la configuración: Caso 2

* Vamos a mirar el log de Sentinel del nodo `maestro`:

```bash
5248:X 28 Feb 2020 15:22:11.632 # +sdown master cursoredis 192.168.157.148 6379
5248:X 28 Feb 2020 15:22:11.698 # +odown master cursoredis 192.168.157.148 6379 #quorum 3/2
5248:X 28 Feb 2020 15:22:11.698 # +new-epoch 4
5248:X 28 Feb 2020 15:22:11.699 # +try-failover master cursoredis 192.168.157.148 6379
5248:X 28 Feb 2020 15:22:11.701 # +vote-for-leader b7e03c1ddaca2b28a496a6e585c02340ed1e25ff 4
5248:X 28 Feb 2020 15:22:11.701 # d7e03c1ddaca2b28a496a6e585c02340ed1e25ff voted for d7e03c1ddaca2b28a496a6e585c02340ed1e25ff 4
5248:X 28 Feb 2020 15:22:11.704 # c7e03c1ddaca2b28a496a6e585c02340ed1e25ff voted for d7e03c1ddaca2b28a496a6e585c02340ed1e25ff 4
5248:X 28 Feb 2020 15:22:12.531 # +config-update-from sentinel d7e03c1ddaca2b28a496a6e585c02340ed1e25ff 192.168.157.148 26379 @ cursoredis 192.168.157.148 6379
5248:X 28 Feb 2020 15:22:12.531 # +switch-master cursoredis 192.168.157.148 6379 192.168.157.144 6379
5248:X 28 Feb 2020 15:22:12.532 * +slave slave 192.168.157.147:6379 192.168.157.147 6379 @ cursoredis 192.168.157.144 6379
5248:X 28 Feb 2020 15:22:12.532 * +slave slave 192.168.157.148:6379 192.168.157.148 6379 @ cursoredis 192.168.157.144 6379
5248:X 28 Feb 2020 15:22:42.556 # +sdown slave 192.168.157.148:6379 192.168.157.148 6379 @ cursoredis 192.168.157.144 6379
```

^^^^^^

#### 💻️ Probando la configuración: Caso 2

* Vemos que tras terminar el proceso de votación, el antiguo maestro ha recuperado su rol y que solo tenemos una réplica:

```redis-cli
redis-cli (maestro) > INFO Replication
# Replication
role:master
connected_slaves:1
slave0:ip=192.168.157.147,port=6379,state=online,offset=2045298,lag=0
master_replid:f407b8c6983bcc6b1c188c2ad9bcbfd21fe13a43
master_replid2:bee1727aa21f3607c6d24bdbb9272ac78769da2f
... 
```

^^^^^^

#### 💻️ Probando la configuración: Caso 2

* Si volvemos a levantar la réplica:

```bash
(replica2) > rc-service redis start
```

* Recuperamos la réplica:

```redis-cli
redis-cli (maestro) > INFO Replication
# Replication
role:master
connected_slaves:2
slave0:ip=192.168.157.147,port=6379,state=online,offset=2092338,lag=1
slave1:ip=192.168.157.148,port=6379,state=online,offset=2092338,lag=0
master_replid:f407b8c6983bcc6b1c188c2ad9bcbfd21fe13a43
master_replid2:bee1727aa21f3607c6d24bdbb9272ac78769da2f
... 
```

^^^^^^

#### Probando la configuración

Vamos a probar la configuración de dos maneras:

* ✅ Caso 1: Activando un failover manualmente
* ✅ Caso 2: Apagando el proceso Redis en el maestro
* Caso 3: Tiramos Redis en las dos réplicas
* Caso 4: Tiramos un proceso Sentinel
* Caso 5: Tiramos dos procesos Sentinel 

^^^^^^

#### 💻️ Probando la configuración: Caso 3

* Nos aseguramos de que tenemos el cluster con el maestro y las dos réplicas funcionando
* Mirando la configuración de Redi, buscamos el maestro y ejecutamos el siguiente comando:

```redis-cli
redis-cli (maestro) > CONFIG SET MIN-SLAVES-TO-WRITE 1
OK 
```

* Estamos configurando el maestro para que **no acepte operaciones de escritura a menos que
  disponga de una réplica activa 

^^^^^^

#### 💻️ Probando la configuración: Caso 3

* Nos conectamos a las dos réplicas y las paramos:

```redis-cli
redis-cli (replica1) > SHUTDOWN
OK 
```

```redis-cli
redis-cli (replica2) > SHUTDOWN
OK 
```

^^^^^^

#### 💻️ Probando la configuración: Caso 3

* Nos conectamos al maestro e intentamos crear una clave nueva:

```redis-cli
redis-cli (maestro) > SET clave valor
(error) NOREPLICAS Not enough good replicas to write. 
```
^^^^^^

#### 💻️ Probando la configuración: Caso 3

* Volvemos a levantar las dos réplicas

```bash
(replica1) > rc-service redis start
```


```bash
(replica2) > rc-service redis start
```

^^^^^^

#### Probando la configuración

Vamos a probar la configuración de dos maneras:

* ✅ Caso 1: Activando un failover manualmente
* ✅ Caso 2: Apagando el proceso Redis en el maestro
* ✅ Caso 3: Tiramos Redis en las dos réplicas
* Caso 4: Tiramos un proceso Sentinel
* Caso 5: Tiramos dos procesos Sentinel 

^^^^^^

#### 💻️ Probando la configuración: Caso 4

* Nos conectamos al Sentinel del maestro:

```bash
(maestro) > redis-cli -h 192.168.157.144 -p 26379
```

^^^^^^

#### 💻️ Probando la configuración: Caso 4

```redis-cli
sentinel (maestro) > SHUTDOWN
not connected > 
```

^^^^^^

#### 💻️ Probando la configuración: Caso 4

* Estando el proceso Sentinel del maestro parado, paramos el proceso Redis del maestro
  para que se inicie el failover
  
```bash
(maestro) > redis-cli -h 192.168.157.144
```

```redis-cli
redis-cli (maestro) > SHUTDOWN
not connected > 
```

  
^^^^^^

#### 💻️ Probando la configuración: Caso 4

* Debido a que tenemos dos procesos sentinel activos todavía, es posible
  renegociar quién es el nuevo maestro
* En este caso, ha pasado a ser la `replica2`:

```redis-cli
redis-cli (replica2) > INFO Replication
# Replication
role:master
connected_slaves:1
slave0:ip=192.168.157.148,port=6379,state=online,offset=2619097,lag=0
master_replid:329f4c8d8537667dbe5a842dd1cf19eb1bb2cc99
master_replid2:f407b8c6983bcc6b1c188c2ad9bcbfd21fe13a43 
```

* Mantenemos el cluster en este estado para hacer el siguiente caso  

^^^^^^

#### Probando la configuración

Vamos a probar la configuración de dos maneras:

* ✅ Caso 1: Activando un failover manualmente
* ✅ Caso 2: Apagando el proceso Redis en el maestro
* ✅ Caso 3: Tiramos Redis en las dos réplicas
* ✅ Caso 4: Tiramos un proceso Sentinel
* Caso 5: Tiramos dos procesos Sentinel 

^^^^^^

#### 💻️ Probando la configuración: Caso 5

* Levantamos de nuevo el proceso Redis del maestro **sin levantar el proceso Sentinel**

```bash
(maestro) > rc-service redis restart
```

^^^^^^

#### 💻️ Probando la configuración: Caso 5

* En el nuevo maestro (`replica2` en nuestro caso) tiramos el proceso Sentinel 
  **y mantenemos el proces Redis activo**
  
```redis-cli
sentinel (replica2) > SHUTDOWN
not connected >
```  

^^^^^^

#### 💻️ Probando la configuración: Caso 5

* Si ahora tiramos el maestro (`replica2` en nuestro caso) **No se iniciará un proceso de failover
  por falta de quorum**. No existe suficientes procesos Sentinel activos para 
  llegar a un acuerdo sobre quién será el nuevo maestro
  
* **Nuestro cluster se ha quedado sin maestro**   

^^^^^^

#### Probando la configuración

Vamos a probar la configuración de dos maneras:

* ✅ Caso 1: Activando un failover manualmente
* ✅ Caso 2: Apagando el proceso Redis en el maestro
* ✅ Caso 3: Tiramos Redis en las dos réplicas
* ✅ Caso 4: Tiramos un proceso Sentinel
* ✅ Caso 5: Tiramos dos procesos Sentinel 

😎
