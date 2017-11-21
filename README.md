# Parcial # 2 - Sistemas Distribuidos
## Nombre: Laura Aragón
## Código: A00268532
## Perfil Github: https://github.com/LauraAragon

### Objetivo
El objetivo del Parcial # 2 de Sistemas Distribuidos es realizar de forma autónoma el aprovisionamiento automático de infraestructura, permitiendo el diagnóstico y la ejecución de forma autónoma de las acciones necesarias para lograr infraestructuras estables integrando servicios que se ejecutan sobre nodos distintos. Todo esto utilizando Docker y docker-compose

### Descripción
Este parcial se realizará utilizando el stack ELK, el cual contiene las siguientes herraientas de la empresa Elastic: Elasticsearch, Logstash y Kibana. Estas tres herramientas fueron utilizadas también en el parcial # 1, por lo tanto ya se esta familiarizado con las funciones de cada una.
El ambiente a aprovisionar está compuesto por:
- Un contenedor encargado de almacenar logs por medio de la aplicación Elasticsearch.
- Un contenedor con la herramienta de visualización Kibana.
- Un contenedor web.
- Un contenedor encargado de realizar la conversión de logs utilizando la aplicación Fluentd.

### 1. Comandos necesarios para el aprovisionamiento de los servicios solicitados.

#### 1.1.1 Instalación e inicio del servicio Elasticsearch

Creación de un directorio para almacenar todo el Stack:
```bash
mkdir -p /Users/amyth/installs/efk
```

Instalación y prueba de Elasticsearch:
```bash
sudo apt-get update
sudo apt-get install openjdk-7-jre

java -version
```

Revisión de la configuración de Java. La configuración necesaria para el correcto funcionamiento del examen se muestra a continuación:
```bash
java version "1.7.0_75"
Java(TM) SE Runtime Environment (build 1.7.0_75-b13)
Java HotSpot(TM) 64-Bit Server VM (build 24.75-b04, mixed mode)
```

```bash
tar -xzvf elasticsearch-2.1.0.tar.gz
mv elasticsearch-2.1.0 ~/installs/efk/

cd ~/installs/efk/elasticsearch-2.1.0
./bin/elasticsearch

or

cd ~/installs/efk/elasticsearch-2.1.0
./bin/elasticsearch -d
```

Una vez iniciado el servicio de Elasticsearch, se realiza una prueba de funcionamiento ingresando a localhost:9200 por medio del navegador, a continuación se muestra el resultado que se debe obtener:
```json
{
  "name" : "Cerise",
  "cluster_name" : "elasticsearch",
  "version" : {
    "number" : "2.1.0",
    "build_hash" : "72cd1f1a3eee09505e036106146dc1949dc5dc87",
    "build_timestamp" : "2015-11-18T22:40:03Z",
    "build_snapshot" : false,
    "lucene_version" : "5.3.1"
  },
  "tagline" : "You Know, for Search"
}
```

#### 1.1.2 Instalación e inicio del servicio Kibana
Para realizar la instalación de Kibana es necesario descargar el archivo .zip del siguiente link: https://www.elastic.co/downloads/kibana, moverlo a la carpeta *~installs/efk* y descomprimirlo como se indica a continuación:
```bash
mv ~/Downloads/kibana-4.3.0-darwin-x64.tar.gz ~/installs/efk
cd ~/installs/efk
tar -xzvf kibana-4.3.0-darwin-x64.tar.gz
```

Inicio del servicio Kibana:
```bash
cd kibana-4.3.0-darwin-x64
./bin/kibana
```

Para realizar la prueba de funcionamiento, se ingresa a http://0.0.0.0:5601 utilizando el navegador y debe visualizarse el dashboard de Kibana.

#### 1.1.3 Instalación e inicio del servicio Fluentd
Fluentd cuenta con un script que automatiza el proceso de instalación, este se encuentra disponible para:
- debian:Â Jessie, Wheezy and Squeeze.
- ubuntu: Trusty, Precise and Lucid
Para obtener el script se utilizan los siguientes comandos, dependiendo del sistema operativo:
```bash
## Debian Jessie
curl -L https://toolbelt.treasuredata.com/sh/install-debian-jessie-td-agent2.sh | sh
 
## Debian Squeeze
curl -L https://toolbelt.treasuredata.com/sh/install-debian-squeeze-td-agent2.sh | sh

## Debian Wheezy
curl -L https://toolbelt.treasuredata.com/sh/install-debian-wheezy-td-agent2.sh | sh

## Ubuntu Trusty
curl -L https://toolbelt.treasuredata.com/sh/install-ubuntu-trusty-td-agent2.sh | sh

## Ubuntu Lucid
curl -L https://toolbelt.treasuredata.com/sh/install-ubuntu-lucid-td-agent2.sh | sh

## Ubuntu Precise
curl -L https://toolbelt.treasuredata.com/sh/install-ubuntu-precise-td-agent2.sh | sh
```

Tras obtener el script se inicia el td-agent:
```bash
/etc/init.d/td-agent restart

#Asegurarse que el td-agent está corriendo	
/etc/init.d/td-agent status
```

#### 1.2. Proceso de integración de Elasticsearch, Fluentd y Kibana stack
1.2.1 Obtencion de los plugins requeridos por Fluentd:
```bash
udo apt-get install make libcurl4-gnutls-dev --yes
sudo /opt/td-agent/embedded/bin/fluent-gem install fluent-plugin-elasticsearch
sudo /opt/td-agent/embedded/bin/fluent-gem install fluent-plugin-record-reformer
```

1.2.2 Envío de la información desde Fluentd a Elasticsearch
Se debe abrir el archivo *etc/td-agent/td-agent.conf* y reemplazar la configuración existente con la configuración mostrada a continuación:
```html
<source>
    type syslog
    port 5140
    tag  system
</source>
 
<match system.*.*>
    type record_reformer
    tag efkl
    facility ${tag_parts[1]}
    severity ${tag_parts[2]}
</match>
 
<match efkl>
    type copy
    <store>
       type elasticsearch
       logstash_format true
       flush_interval 15s
    </store>
</match>
```

1.2.3 Inicio del servicio fluentd mediante los siguientes comandos, dependiendo del sistema operativo:
```bash
## Ubuntu
sudo service td-agent start
 
## Mac OS X
sudo launchctl load /Library/LaunchDaemons/td-agent.plist
```

1.2.4 Indicar a syslog / rsyslog que debe transmitir los datos de registro a fluentd.
Para esto se debe abrir el archivo de configuración de syslog mediante el siguiente comando, dependiendo del sistema operativo:
```bash
## Ubuntu
sudo vim /etc/rsyslog.conf
 
## Mac OS X
sudo vim /etc/rsyslog.conf
```

Y agregar la línea que se presentará a continuación al archivo. Esta línea corresponde a la instrucción del reenvío de los datos de registro desde syslog al host 127.0.0.1 en el puerto 5140:
```bash
*.*                             @127.0.0.1:5140
```

1.2.5 Reinicio del servicio
```bash
curl -XPUT 'http://localhost:9200/kibana/' -d '{"index.mapper.dynamic": true}'
```

1.2.6 Configuraciones dashboard Kibana
A continuación es necesario dirigirse al dashboard de kibana utilizando un navegador (http://0.0.0.0:5601), elegir la pestaña de configuración e ingresar **kibana*/** en el campo “index name or pattern”. Adicionalmente, es necesario desmarcar la opción "Index contains time-based events" y finalizar dando click en "crear".

1.2.7 Prueba de funcionamiento
Finalmente se debe ingrsar a la pestaña **Discover**, donde se deben visualizar los logs del syslog.

#### 2. Archivos Dockerfile
```Dockerfile
# fluentd/Dockerfile
FROM fluent/fluentd:v0.12.40-debian
RUN ["gem", "install", "fluent-plugin-elasticsearch", "--no-rdoc", "--no-ri", "--version", "1.9.2"]
```

#### 3. Archivos para el despliegue de la infraestructura.
```yml
version: '2'
services:
  web:
    image: httpd
    ports:
      - "80:80"
    links:
      - fluentd
    logging:
      driver: "fluentd"
      options:
        fluentd-address: localhost:24224
        tag: httpd.access

  fluentd:
    build: ./fluentd
    volumes:
      - ./fluentd/conf:/fluentd/etc
    links:
      - "elasticsearch"
    ports:
      - "24224:24224"
      - "24224:24224/udp"

  elasticsearch:
    image: elasticsearch
    expose:
      - 9200
    ports:
      - "9200:9200"

  kibana:
    image: kibana
    links:
      - "elasticsearch"
    ports:
      - "5601:5601"
```

#### 4. Estructura de carpetas y archivos necesarios para el aprovisionamiento.

4.1 Estructura de archivos:
```
sd-exam2
    ├── A00068012
    │   ├── docker-compose.yml
    │   ├── fluentd
    │   │   ├── conf
    │   │   │   └── fluent.conf
    │   │   └── Dockerfile
    │   └── README.md
    └── README.md
```

4.2 Fluent.conf:
```c
# fluentd/conf/fluent.conf
<source>
  @type forward
  port 24224
  bind 0.0.0.0
</source>
<match *.**>
  @type copy
  <store>
    @type elasticsearch
    host elasticsearch
    port 9200
    logstash_format true
    logstash_prefix fluentd
    logstash_dateformat %Y%m%d
    include_tag_key true
    type_name access_log
    tag_key @log_name
    flush_interval 1s
  </store>
  <store>
    @type stdout
  </store>
</match>
```

#### 5. Pruebas del funcionamiento
5.1 Contenedores:
Los puertos en los que corre cada uno de los servicios son los siguientes:
- ElasticSearch: 9200
- Servidor Web: 80
- Kibana: 5601
- Fluentd: 24224

![][1]

5.2 Servidor Web:
La única funcionalidad de este servidor es recibir peticiones y enviarlas al contenedor Fluentd.

![][2]

5.3 ElasticSearch
Fluentd envía los logs que recibe al contenedor de Elasticsearch, éste funciona como una base de datos almacenándolos, éstos logs pueden ser accedidos por medio de peticiones a ciertos endpoints

![][3]

5.4 Kibana:
Su función es permitir visualizar los logs ubicados en el contenedor de Elasticsearch.

![][4]

[1]: images/Containers.png
[2]: images/Webserver.png
[3]: images/Elastic.png
[4]: images/kibana.png