### Configuración de Sentinel

![](/slides/images/master_slaves/master_slaves.001.jpeg)<!-- .element: style="height: 50vh" -->

notes:

Esta es la infraestructura sobre la que vamos a instalar Sentinel. Estas tres instancias
ya deberías tenerlas correctamente configuradas tras el ejercicio del módulo 4.

^^^^^^

#### Configuración: consideraciones previas

* Se necesitan al menos tres procesos Sentinel para disponer de un despliegue robusto
* Las tres instancias de sentinel deben estar en máquinas virtuales o físicas que puedan 
  fallar de manera independiente (poner dos o más instancias en un mismo servidor puede causar problemas)
  

notes: 

Si ponemos dos o más instancias sobre el mismo servidor y este se cae, puede darse el caso
de que nos quedemos con un sólo nodo en el cluster y que Sentinel no sea capaz de llevar a
cabo el Failover por falta de quorum.

Por este mismo motivo, si estás utilizando máquinas virtuales conviene que las instancias estén
en zonas de disponibilidad diferentes para que no se caigan todas juntas.


^^^^^^

#### Configuración: consideraciones previas

* El uso de Sentinel y Redis **no garantiza que las operaciones de escritura se persistan en caso de fallo porque Redis utiliza replicación asíncrona**. 
  Hay maneras de desplegar Sentinel en las que las ventanas en las que podemos perder información queden restringidas
  a determinados momentos, pero no podemos garantizar nada más.

^^^^^^

#### Configuración: consideraciones previas

* Necesitas que tus clientes soporten Sentinel para que se les pueda notificar de los cambios
  en el cluster que puedan ocurrir. **No todos los clientes lo soportan así que por favor, sed majos y habladlo con los programadores**

^^^^^^

#### Configuración: consideraciones previas

* Testad vuestra configuración... de forma extensiva y si es posible testar en producción mejor que mejor 
  (ya sabes, tirar un nodo en un momento de máximo tráfico)

![chuck_norris](/slides/images/chuck_norris.jpg)

^^^^^^

#### Configuración: consideraciones previas

* Cuidado si usáis Docker o cualquier otro sistema que implemente NAT o _port remapping_ ya que estas configuraciones
  pueden impedir que los demonios Sentinel se auto-localicen solos

^^^^^^

#### Configuración: consideraciones previas

* Para que Sentinel se levante **tiene que tener permiso de escritura sobre el fichero de configuración**

notes:

En la máquina virtual que estamos utilizando, el gestor de paquetes que se utilizó para instalar redis
se encargó de que este requisito ya se cumpla.

^^^^^^

#### 💻️ Configuración

La configuración mínima para que Sentinel funcione es la siguiente:

```
port 26379
bind 192.168.157.14[X]
dir /tmp 
sentinel monitor cursoredis 192.168.157.144 6379 2 
sentinel down-after-milliseconds cursoredis 30000 
sentinel failover-timeout cursoredis 180000 
sentinel parallel-syncs cursoredis 1 
```

notes:

Debemos especificar:

* El puerto en el que Sentinel escucha
* La carpeta para los ficheros temporales necesarios para su ejecución
* El nombre del maestro, su IP, el puerto en el que está redis y el quorum (2)
* `down-after-milliseconds` Tiempo que el master debe estar sin responder para que el proceso
  Sentinel considere que está caído
* `parallel-syncs` Número de instancias que pueden solicitar una resincronización al maestro de manera simultánea   
* `failover-timeout` Tiempo de espera que Sentinel utiliza de diferentes maneras a la hora
  de llevar a cabo el proceso de failover 
 
Editar el fichero `/etc/sentinel.conf` y modificar la configuración
en las tres máquinas.


^^^^^^

#### 💻️ Configuración

⚠️ Si has copiado las máquinas virtuales para generar las réplicas, es necesario
editar el fichero `/etc/sentinel.conf` y cambiar el ID.

```
sentinel myid b7e03c1ddaca2b28a496a6e585c02340ed1e25ff
```

notes:

Si no modificas este ID y los tres procesos Sentinel tienen el mismo valor, el sistema no funcionará.

^^^^^^

#### 💻️ Configuración
 
Una vez configurado Sentinel en cada máquina, empezamos levantándolo en el maestro:

```bash 
(maestro)  > rc-service redis-sentinel start
```

Si miramos el log de Sentinel (`/var/log/redis/sentinel.log`) en el maestro veremos:

```
5097:X 28 Feb 2020 12:48:08.556 # oO0OoO0OoO0Oo Redis is starting oO0OoO0OoO0Oo
5097:X 28 Feb 2020 12:48:08.556 # Redis version=5.0.7, bits=64, commit=bed89672, modified=0, pid=5097, just started
5097:X 28 Feb 2020 12:48:08.556 # Configuration loaded
5097:X 28 Feb 2020 12:48:08.557 * Running mode=sentinel, port=26379.
5097:X 28 Feb 2020 12:48:08.557 # Sentinel ID is b7e03c1ddaca2b28a496a6e585c02340ed1e25ff
5097:X 28 Feb 2020 12:48:08.557 # +monitor master cursoredis 192.168.157.144 6379 quorum 2
```

^^^^^^

#### 💻️ Configuración

Ahora lo levantamos en la primera réplica:

```bash 
(replica1)  > rc-service redis-sentinel start
```

Si miramos el log de Sentinel (`/var/log/redis/sentinel.log`) en la réplica veremos:

```
4730:X 28 Feb 2020 12:53:16.410 # oO0OoO0OoO0Oo Redis is starting oO0OoO0OoO0Oo
4730:X 28 Feb 2020 12:53:16.410 # Redis version=5.0.7, bits=64, commit=bed89672, modified=0, pid=4730, just started
4730:X 28 Feb 2020 12:53:16.410 # Configuration loaded
4730:X 28 Feb 2020 12:53:16.411 * Running mode=sentinel, port=26379.
4730:X 28 Feb 2020 12:53:16.411 # Sentinel ID is b7e03c1ddaca2b28a496a6e585c02340ed1e25ff
4730:X 28 Feb 2020 12:53:16.411 # +monitor master cursoredis 192.168.157.144 6379 quorum 2
4730:X 28 Feb 2020 12:53:16.414 * +slave slave 192.168.157.147:6379 192.168.157.147 6379 @ cursoredis 192.168.157.144 6379
4730:X 28 Feb 2020 12:53:16.415 * +sentinel sentinel c7e03c1ddaca2b28a496a6e585c02340ed1e25ff 192.168.157.147 26379 @ cursoredis 192.168.157.144 6379
```

^^^^^^

#### 💻️ Configuración

Por último, lo levantamos en la segunda réplica:

```bash 
(replica2)  > rc-service redis-sentinel start
```

^^^^^^

#### 💻️ Configuración

Si abrimos los ficheros de configuración de Sentinel, veremos que 
el propio demonio ha añadido varias líneas al final de cada uno de ellos:

```
# Generated by CONFIG REWRITE (maestro)
protected-mode no
sentinel known-sentinel cursoredis 192.168.157.147 26379 c7e03c1ddaca2b28a496a6e585c02340ed1e25ff
sentinel known-sentinel cursoredis 192.168.157.148 26379 d7e03c1ddaca2b28a496a6e585c02340ed1e25ff
sentinel current-epoch 2
```

^^^^^^

#### 💻️ Configuración

```
# Generated by CONFIG REWRITE (replica1)
protected-mode no
sentinel known-replica cursoredis 192.168.157.148 6379
sentinel known-replica cursoredis 192.168.157.147 6379
sentinel known-sentinel cursoredis 192.168.157.148 26379 d7e03c1ddaca2b28a496a6e585c02340ed1e25ff
sentinel known-sentinel cursoredis 192.168.157.144 26379 b7e03c1ddaca2b28a496a6e585c02340ed1e25ff
sentinel current-epoch 2
```

^^^^^^

#### 💻️ Configuración

```
# Generated by CONFIG REWRITE (replica2)
protected-mode no
sentinel known-replica cursoredis 192.168.157.148 6379
sentinel known-replica cursoredis 192.168.157.147 6379
sentinel known-sentinel cursoredis 192.168.157.144 26379 b7e03c1ddaca2b28a496a6e585c02340ed1e25ff
sentinel known-sentinel cursoredis 192.168.157.147 26379 d7e03c1ddaca2b28a496a6e585c02340ed1e25ff
sentinel current-epoch 2
```

^^^^^^

#### 💻️ Configuración

Para verificar la configuración, nos conectamos a Sentinel usando `redis-cli`...

```bash
(replica1) > redis-cli -h 192.168.157.147 -p 26379
                                          ^^^^^^^^
  
```

^^^^^^

#### 💻️ Configuración

...y usamos el comando [`INFO`](https://redis.io/commands/info) para mostrar por pantalla
la configuración del proceso:

```redis-cli
redis-cli > > INFO SENTINEL
# Sentinel
sentinel_masters:1
sentinel_tilt:0
sentinel_running_scripts:0
sentinel_scripts_queue_length:0
sentinel_simulate_failure_flags:0
master0:name=cursoredis,status=ok,address=192.168.157.144:6379,slaves=2,sentinels=3 
```

^^^^^^

#### Configuración

Sentinel se conecta cada 10 segundos a cada nodo de la red y 
ejecuta en el comando [`INFO Replication`](https://redis.io/commands/info)
para tener en todo momento una información completa de la topología de la red

^^^^^^

#### Configuración
 
Además, para detectar y comunicarse con otros procesos Sentinel, cada 2 segundos
cada proceso Sentinel publica en el canal `__sentinel__:hello` un mensaje acerca de su estado y como considera que está
el maestro (caído o levantado)

^^^^^^

#### 💻️ Configuración: `__sentinel__:hello`

```redis-cli
redis-cli > SUBSCRIBE __sentinel__:hello
1) "subscribe"
2) "__sentinel__:hello"
3) (integer) 1
1) "message"
2) "__sentinel__:hello"
3) "192.168.157.144,26379,b7e03c1ddaca2b28a496a6e585c02340ed1e25ff,2,cursoredis,192.168.157.144,6379,2"
1) "message"
2) "__sentinel__:hello"
3) "192.168.157.144,26379,b7e03c1ddaca2b28a496a6e585c02340ed1e25ff,2,cursoredis,192.168.157.144,6379,2"
1) "message"
2) "__sentinel__:hello"
3) "192.168.157.148,26379,d7e03c1ddaca2b28a496a6e585c02340ed1e25ff,2,cursoredis,192.168.157.144,6379,2"
1) "message"
2) "__sentinel__:hello"
3) "192.168.157.148,26379,d7e03c1ddaca2b28a496a6e585c02340ed1e25ff,2,cursoredis,192.168.157.144,6379,2" 
```

notes:

Podemos ejecutar este comando desde cualquier nodo activo de nuestro cluster.