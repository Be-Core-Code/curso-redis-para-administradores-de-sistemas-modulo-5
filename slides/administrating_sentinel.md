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
### 💻️ Prácticas

* Práctica 1: Enviar un email cuando ocurren los eventos de Sentinel
* Práctica 2: Reconfigurar el maestro tras un failover

^^^^^^

### 💻️ Práctica 1: Enviar un email

⛔ 

Configuración SMTP

notes:

En este punto:

* Os daremos los parámetros para enviar correos electrónicos por STMP
* Autorizaremos vuestros emails para que podáis enviar emails

Si estás repasando las diapositivas después del curso, este es un buen momento para que te hagas con los parámetros
de envío de tu proveedor de correo electrónico.  

^^^^^^

#### 💻️ Práctica 1: Enviar un email

* Hemos preparado un script en Python para enviar este correo electrónico. Puedes encontrarlo en 
  [este repositorio](https://github.com/Be-Core-Code/curso-redis-para-administradores-de-sistemas)
* Dentro de la máquina virtual, ya hemos clonado este repositorio en la carpeta
  `/usr/local/curso-redis-para-administradores-de-sistemas/`
* Puedes ejecutar `git pull` para asegurarte de que tienes la última versión

```bash
> cd /usr/local/curso-redis-para-administradores-de-sistemas/
> git pull 
``` 
^^^^^^

#### 💻️ Práctica 1: Enviar un email

* Para que funcione, el script debe estar disponible en todas las máquinas virtuales y terner permiso de ejecución

notes:

Si has copiado la máquina virtual que facilitamos para este curso, ya dispondrás del repositorio en la ruta adecuada.

Si has creado tu propia máquina virtual, asegúrate de clonar 
[el repositorio](https://github.com/Be-Core-Code/curso-redis-para-administradores-de-sistemas) y, si lo clonas en una ruta diferente
a la que estamos usando en las diapositivas, recuerda cambiarla cuando ejecutes los comandos.

^^^^^^

#### 💻️ Práctica 1: Enviar un email
 
* Vamos a intentar configurar el script conectándonos a Sentinel:

```bash 
(maestro) > redis-cli -h 192.168.157.144 -h 26379 
```

```redis-cli
sentinel > SENTINEL SET cursoredis notification-script cursoredis /usr/local/curso-redis-para-administradores-de-sistemas/modulo-5/envia_notificacion.py
(error) ERR Reconfiguration of scripts path is denied for security reasons. 
Check the deny-scripts-reconfig configuration directive in your Sentinel configuration 
```
> 

notes:

En la configuración por defecto de Sentinel, no se permite modificar la ruta a los scripts desde el cliente. Esta
es una buena práctica desde el punto de vista de la seguridad así que en lugar de hacerlo por comandos modificaremos
el fichero de configuración.
 
^^^^^^

#### 💻️ Práctica 1: Enviar un email

* Editamos el fichero de configuración de sentinel en uno de los procesos (en este caso en el maestro) y recargamos la configuración

```bash
# /etc/sentinel.conf
sentinel notification-script cursoredis /usr/local/curso-redis-para-administradores-de-sistemas/modulo-5/envia_notificacion.py
```

```bash
maestro > rc-service redis-sentinel restart
```

^^^^^^

#### 💻️ Práctica 1: Enviar un email

* Para probar que funciona, podemos tirar una de las réplicas

```bash 
(replica2) > rc-service redis stop
```
 
^^^^^^
#### 💻️ Prácticas

* ✅ Práctica 1: Enviar un email cuando ocurren los eventos de Sentinel
* Práctica 2: Reconfigurar el maestro tras un failover

^^^^^^

#### 💻️ Práctica 2: Reconfiguración del maestro

* Redis permite ejecutar un script cuando tras un failover
* Esta opción es muy útil cuando queremos que la configuración del maestro sea diferente que la de las réplicas

^^^^^^

#### 💻️ Práctica 2: Reconfiguración del maestro

* Para esta práctica, buscamos que
  * El nodo que esa maestro **no** debe generar un fichero RDB
  * Un nodo que sera una réplica **sí** debe generar un fichero RDB
    
^^^^^^

#### 💻️ Práctica 2: Reconfiguración del maestro

* Hemos preparado un script en bash para reconfigurar el maestro. Puedes encontrarlo en 
  [este repositorio](https://github.com/Be-Core-Code/curso-redis-para-administradores-de-sistemas)
* Dentro de la máquina virtual, ya hemos clonado este repositorio en la carpeta
  `/usr/local/curso-redis-para-administradores-de-sistemas/`
* Puedes ejecutar `git pull` para asegurarte de que tienes la última versión

```bash
> cd /usr/local/curso-redis-para-administradores-de-sistemas/
> git pull 
``` 

notes:

Si has copiado la máquina virtual que facilitamos para este curso, ya dispondrás del repositorio en la ruta adecuada.

Si has creado tu propia máquina virtual, asegúrate de clonar 
[el repositorio](https://github.com/Be-Core-Code/curso-redis-para-administradores-de-sistemas) y, si lo clonas en una ruta diferente
a la que estamos usando en las diapositivas, recuerda cambiarla cuando ejecutes los comandos.


^^^^^^

#### 💻️ Práctica 2: Reconfiguración del maestro

* Para que funcione, el script debe estar disponible en todas las máquinas virtuales y terner permiso de ejecución

^^^^^^

#### 💻️ Práctica 2: Reconfiguración del maestro

* Editamos la configuración **del maestro y de las réplicas** y añadimos la siguiente línea:

```bash
# /etc/sentinel.conf
sentinel client-reconfig-script cursoredis /usr/local/curso-redis-para-administradores-de-sistemas/modulo-5/configura_rdb.bash
```

^^^^^^

#### 💻️ Práctica 2: Reconfiguración del maestro

* Parámetros que recibe el script
  * `master-name` 
  * `role` que es `leader` o `observer`
  * `state` Actualmente es siempre `failover`
  * `from-ip` IP del maestro antes del failover
  * `from-port` Puerto del maestro antes del failover
  * `to-ip` IP del maestro después del failover
  * `to-port`Puerto del maestro después del failover

^^^^^^

#### 💻️ Práctica 2: Reconfiguración del maestro

* Reiniciamos Sentinel en todos los nodos

```bash
(maestro)  > rc-service redis-sentinel restart
(replica1) > rc-service redis-sentinel restart
(replica2) > rc-service redis-sentinel restart
```

^^^^^^

#### 💻️ Práctica 2: Reconfiguración del maestro

* Vamos al actual maestro y desactivamos la creación del fichero RDB

```redis-cli
redis-cli (maestro) > CONFIG SET SAVE ""
OK 
```

^^^^^^

#### 💻️ Práctica 2: Reconfiguración del maestro

* Paramos Redis y Sentinel en el maestro:

```bash
(maestro) > rc-service redis stop && rc-service redis-sentinel stop
```

^^^^^^

#### 💻️ Práctica 2: Reconfiguración del maestro

* Pasados 30 segundos (según tenemos configurado en nuestro fichero `/etc/sentinel.conf`) podemos verificar que el
  nuevo maestro no tiene activado el parámetro `save`
  
```redis-cli
redis-cli (replica1) > info replication
role:master
....
redis-cli (replica1) > config get save
1) "save"
2) ""
```
 
^^^^^^

#### 💻️ Práctica 2: Reconfiguración del maestro

* Si levantamos de nuevo el antiguo maestro, podemos verificar que si se mantiene como réplica tendrá configurado 
  correctamente el parámetro `save`
  
```bash
(maestro) > rc-service redis start && rc-service redis-sentinel start
```   
```redis-cli
redis-cli (maestro) > info replication
role:slave
....
redis-cli (maestro) config get save
1) "save"
2) "900 1 300 10 60 10000"
```

^^^^^^
### 💻️ Prácticas

* ✅ Práctica 1: Enviar un email cuando ocurren los eventos de Sentinel
* ✅ Práctica 2: Reconfigurar el maestro tras un failover

😎
