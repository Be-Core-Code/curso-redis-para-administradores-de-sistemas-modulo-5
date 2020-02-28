### Comandos utilizados en este módulo

* [`INFO`](https://redis.io/commands/info)
* [`SHUTDOWN`](https://redis.io/commands/shutdown)
* [`SUBSCRIBE`](https://redis.io/commands/subscribe)
* `SENTINEL FAILOVER`
* `SENTINEL GET-MASTER-ADDR-BY-NAME`
* `SENTINEL MASTERS`
* `SENTINEL SLAVES`
* `SENTINEL SET`

^^^^^^

### Opciones de configuración (sentinel.conf)

* `port`
* `bind`
* `dir` 
* `sentinel monitor` 
* `sentinel down-after-milliseconds` 
* `sentinel failover-timeout` 

^^^^^^

### Opciones de configuración (sentinel.conf)

* `sentinel parallel-syncs` 
* `sentinel known-sentinel`
* `sentinel known-replica`


[Sobre `redis.conf`](https://redis.io/topics/config)

[Sobre `sentinel.conf`](http://download.redis.io/redis-stable/sentinel.conf)