# Sistema IoT de riego inteligente

Sistema de supervisión y control remoto del riego que integra dispositivos ESP32 con una plataforma de servicios desplegada en equipos de borde. La solución permite consultar telemetría, operar electroválvulas desde una interfaz web y ejecutar riegos manuales o temporizados.

## Descripción para CV

Desarrollé un sistema IoT de riego remoto con Node-RED, MQTT y Docker sobre Raspberry Pi y Orange Pi 5. Integré microcontroladores ESP32 programados en C++ para recopilar datos de caudal, temperatura y humedad, y controlar electroválvulas. La solución incluye una interfaz web para supervisión y control manual o temporizado, un broker Mosquitto y persistencia de datos de aplicación y estados en MySQL.

## Objetivo

Desarrollar una solución IoT que reduzca la necesidad de desplazamientos diarios para operar el riego y facilite un uso más eficiente del agua. Para ello, centraliza la comunicación con los dispositivos de campo, la lógica de control, la supervisión web y el almacenamiento de información del sistema.

## Arquitectura

```mermaid
flowchart LR
    usuario["Usuario<br/>Navegador web"]
    internet["Internet / acceso remoto"]
    router["Router ISP"]
    firewall["Firewall Fortinet"]

    subgraph edge["Infraestructura de borde"]
        subgraph hosts["Servidores: Raspberry Pi y Orange Pi 5"]
            subgraph docker["Servicios desplegados con Docker"]
                nginx["Nginx<br/>Proxy inverso"]
                nodered["Node-RED<br/>Panel web y lógica de control"]
                mqtt["Mosquitto<br/>Broker MQTT"]
                mysql["MySQL<br/>Datos de aplicación y estados"]
            end
        end
    end

    subgraph campo["Sistema de riego"]
        sensores["Caudalímetros<br/>Temperatura y humedad DHT22"]
        esp["ESP32<br/>Firmware C++"]
        driver["Etapa de accionamiento<br/>Relé / driver"]
        valvulas["Electroválvulas"]
        sensores -->|Mediciones| esp
        esp --> driver --> valvulas
    end

    usuario --> internet --> router --> firewall --> nginx
    nginx -->|Proxy web| nodered
    nodered <-->|Publicación y suscripción MQTT| mqtt
    nodered <-->|Lectura y escritura| mysql
    mqtt <-->|Telemetría, estados y comandos| esp

    classDef network fill:#eef2ff,stroke:#6366f1,color:#111827
    classDef service fill:#ecfeff,stroke:#0891b2,color:#111827
    classDef field fill:#f0fdf4,stroke:#16a34a,color:#111827
    class usuario,internet,router,firewall network
    class nginx,nodered,mqtt,mysql service
    class sensores,esp,driver,valvulas field
```

### Flujo de supervisión

Los ESP32 envían por MQTT las mediciones de los caudalímetros y de los sensores DHT22. Node-RED recibe los mensajes, presenta la información en la interfaz web y gestiona la información de aplicación y los estados en MySQL.

### Flujo de control

El usuario acciona las electroválvulas desde el panel web de Node-RED, de forma manual o mediante una duración programada. Node-RED publica los comandos en Mosquitto; los ESP32 suscritos los reciben y activan la etapa de accionamiento correspondiente. Los estados y la telemetría pueden volver al panel mediante MQTT.

## Componentes

| Capa | Componentes | Responsabilidad |
| --- | --- | --- |
| Campo | ESP32 programados en C++, caudalímetros, sensores DHT22 y electroválvulas | Medir variables del sistema y ejecutar órdenes de riego. |
| Mensajería | Mosquitto y MQTT | Transportar telemetría, estados y comandos entre Node-RED y los ESP32. |
| Aplicación | Node-RED y funciones JavaScript | Proporcionar el panel web y coordinar el control manual y temporizado. |
| Datos | MySQL | Persistir información de la aplicación y estados del sistema. |
| Acceso | Nginx, firewall Fortinet y router ISP | Encauzar el acceso web a los servicios de la instalación. |
| Despliegue | Docker sobre Raspberry Pi y Orange Pi 5 | Alojar los servicios de la plataforma en infraestructura de borde. |

## Tecnologías

`ESP32` · `C++` · `Node-RED` · `MQTT` · `Eclipse Mosquitto` · `MySQL` · `Nginx` · `Docker` · `Raspberry Pi` · `Orange Pi 5`

> El esquema refleja los componentes descritos para el proyecto. No se especifican aquí el reparto concreto de contenedores entre placas, la configuración de red, los tópicos MQTT ni las métricas de ahorro de agua.
