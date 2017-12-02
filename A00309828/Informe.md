# Examen 3 #

Universidad ICESI

Curso: Sistemas Distribuidos

Docente: Daniel Barragán C.

Tema: Automatización de infraestructura (Vagrant+Chef)

Correo: daniel.barragan at correo.icesi.edu.co


Objetivos

Realizar de forma autónoma el aprovisionamiento automático de infraestructura

Diagnosticar y ejecutar de forma autónoma las acciones necesarias para lograr infraestructuras estables


Prerrequisitos

Docker

Docker-Compose

Contenedores: consul, consul-template, registrator, load balancer (nginx, haproxy)


Descripción

Deberá realizar el aprovisionamiento de un ambiente compuesto por los siguientes elementos: un servidor web con capacidad de escalar a N instancias (puede	emplear	apache+php o crear	un servicio web con el	lenguaje de su preferencia), un balanceador de carga para redireccionar las peticiones a los servidores web.

Tenga en cuenta:

Para el aprovisionamiento deberá usar docker-compose

Emplear una herramienta de descubrimiento de servicio (zookeper, consul, etcd) que permita registrar automáticamente las nuevas instancias de servidores web. 
Las tecnologías de descubrimiento de servicio se componen de agentes y un servidor ó clúster de servidores. 
Los agentes envían información al clúster acerca de los servicios que se ejecutan en las instancias.
El servidor registran los servicios que son anunciados por los agentes para ser consultados por los clientes ú otros servicios
Para evitar ejecutar mas de un servicio por contenedor (agente de consul y servicio web) emplee la aplicación dockerizada registrator (ó una tecnología similar) para registrar los nuevos contenedores ante el servidor de descubrimiento de servicio
Para actualizar la configuración de los archivos de configuración del balanceador de carga y reiniciar el servicio emplee la aplicación consul-template. Consul-template consulta al servidor de consul el estado de los servicios y ante un cambio en ellos, a partir de plantillas, crea nuevamente los archivos de configuración.


Consigne los comandos de linux necesarios para el aprovisionamiento de los servicios solicitados. En este punto no debe incluir archivos tipo Dockerfile solo se requiere que usted identifique los comandos o acciones que debe automatizar ***

```
docker run -d --name=consul -p 8500:8500 consul
```

```
docker run -d \
    --name=registrator \
    --net=host \
    --volume=/var/run/docker.sock:/tmp/docker.sock \
    gliderlabs/registrator:latest \
    -internal \
      consul://localhost:8500
```

```
docker logs registrator
```

```
curl localhost:8500/v1/catalog/services
```

![c1](https://user-images.githubusercontent.com/17281732/33511966-10afe10e-d6f4-11e7-8b61-41c51447aebe.PNG)


Escriba los archivos Dockerfile para cada uno de los servicios solicitados junto con los archivos fuente necesarios. 
Tenga en cuenta consultar buenas prácticas para la elaboración de archivos Dockerfile
DockerFile Haproxy

```
FROM centos:latest

#Instalacion de haproxy
RUN yum -y install wget && yum -y install unzip && yum -y install haproxy

#Instalacion de consul-template
ENV CONSUL_TEMPLATE_VERSION 0.19.3

ADD https://releases.hashicorp.com/consul-template/${CONSUL_TEMPLATE_VERSION}/consul-template_${CONSUL_TEMPLATE_VERSION}_SHA256SUMS /tmp/
ADD https://releases.hashicorp.com/consul-template/${CONSUL_TEMPLATE_VERSION}/consul-template_${CONSUL_TEMPLATE_VERSION}_linux_amd64.zip /tmp/

RUN cd /tmp && \
    sha256sum -c consul-template_${CONSUL_TEMPLATE_VERSION}_SHA256SUMS 2>&1 | grep OK && \
    unzip consul-template_${CONSUL_TEMPLATE_VERSION}_linux_amd64.zip && \
    mv consul-template /bin/consul-template && \
    rm -rf /tmp

WORKDIR /etc/haproxy
ADD haproxy.ctmpl .

```

DockerFile Web
```
FROM python:3.4-alpine
ADD . /code
WORKDIR /code
RUN pip install -r requirements.txt
CMD ["python", "app.py"]

```

Escriba el archivo docker-compose.yml necesario para el despliegue de la infraestructura.
No emplee configuraciones deprecated. Incluya un diagrama general de los componentes empleados

![c2](https://user-images.githubusercontent.com/17281732/33512008-115f264a-d6f5-11e7-9dd7-a14f7bef1872.PNG)

Diagrama
![diagramafinal](https://user-images.githubusercontent.com/17281732/33512040-a424483e-d6f5-11e7-9459-4cbb84025ebc.png)


Incluya evidencias que muestran el funcionamiento de lo solicitado

![evidencia1-docker-compose](https://user-images.githubusercontent.com/17281732/33512061-154afc1a-d6f6-11e7-8032-26fd53c00dfd.png)
![evidencia2-docker-compose](https://user-images.githubusercontent.com/17281732/33512065-218a055c-d6f6-11e7-8a60-c26bdfd67da8.png)
![evidencia2-scale 3](https://user-images.githubusercontent.com/17281732/33512066-22034d04-d6f6-11e7-97af-a1198e3b4130.png)
![evidencia3-scale 3](https://user-images.githubusercontent.com/17281732/33512067-222bbeba-d6f6-11e7-8356-abe5cffa7eb8.png)
![evidencia4-scale 6](https://user-images.githubusercontent.com/17281732/33512069-23474a80-d6f6-11e7-81bb-072e7199b1c9.png)
![evidencia5-scale 6](https://user-images.githubusercontent.com/17281732/33512070-23c05eac-d6f6-11e7-923a-69848a358f5c.png)

Documente algunos de los problemas encontrados y las acciones efectuadas para su solución al aprovisionar la infraestructura y aplicaciones

Problema: comunicar haproxy con consul sin depender de la ip

Solucion: 

Para que haproxy se actualice dinámicamente debe ejecutar consul-template en background. Se pensó ejecutarlo usando la directiva RUN de Dockerfile pero se obtuvieron problemas y además se dependía de la ip con la cual quedara el contenedor de consul.
Por ello, se decidio ejecutar el comando necesario directamente desde docker-compose usando la directiva command

 ![c3](https://user-images.githubusercontent.com/17281732/33512084-7e064db8-d6f6-11e7-92e5-0d503b4a62c3.png)




