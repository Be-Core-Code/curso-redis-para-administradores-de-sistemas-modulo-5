### Administración de Sentinel

En esta sección vamos a ver algunos comandos que nos ayudarán a 
administrar Sentinel

^^^^^^

#### Administración de Sentinel

* `SENTINEL GET-MASTER-ADDR-BY-NAME`: nos devuelve la IP del maestro

```redis-cli
sentinel >  SENTINEL GET-MASTER-ADDR-BY-NAME cursoredis
1) "192.168.157.144"
2) "6379"
```

^^^^^^

#### Administración de Sentinel

* `SENTINEL MASTERS`: Lista los maestros que tengamos configurados en el cluster

```redis-cli
sentinel >  SENTINEL MASTERS
1)  1) "name"
    2) "cursoredis"
    3) "ip"
    4) "192.168.157.144"
    5) "port"
    6) "6379"
    7) "runid"
    8) "8894f9364b87745009fc54ad3227b6ec9af0a94a"
...
   39) "parallel-syncs"
   40) "1"
```

^^^^^^

#### Administración de Sentinel

* `SENTINEL SLAVES`: Lista las réplicas del maestro que le pasemos como argumento

```redis-cli
sentinel > REDIS SLAVES cursoredis
1)  1) "name"
    2) "192.168.157.148:6379"
    3) "ip"
    4) "192.168.157.148"
    5) "port"
    6) "6379" 
...
2)  1) "name"
    2) "192.168.157.147:6379"
    3) "ip"
    4) "192.168.157.147"
    5) "port"
    6) "6379"
```

^^^^^^

#### Administración de Sentinel

* `SENTINEL SET`: Actualiza la configuración de Sentinel

```redis-cli
sentinel > SENTINEL SET cursoredis down-after-milliseconds 2000 
```

^^^^^^

### 💻️ Ejecutar script cuando tiene lugar el failover


