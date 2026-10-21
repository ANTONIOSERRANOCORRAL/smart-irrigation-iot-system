# Sistema IoT de riego inteligente

Sistema de supervisión y control remoto del riego que integra dispositivos ESP32 con una plataforma de servicios desplegada en equipos de borde. La solución permite consultar telemetría, operar electroválvulas desde una interfaz web y ejecutar riegos manuales o temporizados.

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
        esp -->|Señal de control| driver
        driver -->|Apertura / cierre| valvulas
    end

    usuario --> internet --> router --> firewall --> nginx
    nginx -->|Proxy web| nodered
    esp -->|Publica telemetría| mqtt
    mqtt -->|Telemetría suscrita| nodered
    nodered -->|Publica órdenes de riego| mqtt
    mqtt -->|Órdenes suscritas| esp
    nodered <-->|Lectura y escritura| mysql

    classDef network fill:#eef2ff,stroke:#6366f1,color:#111827
    classDef service fill:#ecfeff,stroke:#0891b2,color:#111827
    classDef field fill:#f0fdf4,stroke:#16a34a,color:#111827
    class usuario,internet,router,firewall network
    class nginx,nodered,mqtt,mysql service
    class sensores,esp,driver,valvulas field
```

### Flujo de supervisión

Los caudalímetros y sensores DHT22 entregan sus mediciones a los ESP32. Cada ESP32 publica la telemetría en Mosquitto mediante MQTT; Node-RED se suscribe a esos mensajes y presenta los datos en la interfaz web. Node-RED también lee y guarda en MySQL la información de aplicación y los estados del sistema.

### Flujo de control

El usuario envía órdenes desde la interfaz web, en modo manual o con una duración programada. Nginx deriva la petición a Node-RED, que publica la orden en Mosquitto. El ESP32 suscrito al mensaje recibe la orden y acciona la electroválvula mediante la etapa de control.

## Componentes

| Capa | Componentes | Responsabilidad |
| --- | --- | --- |
| Campo | ESP32 programados en C++, caudalímetros, sensores DHT22 y electroválvulas | Medir variables del sistema y ejecutar órdenes de riego. |
| Mensajería | Mosquitto y MQTT | Transportar telemetría desde los ESP32 a Node-RED y órdenes de riego en sentido inverso. |
| Aplicación | Node-RED y funciones JavaScript | Proporcionar el panel web y coordinar el control manual y temporizado. |
| Datos | MySQL | Persistir información de la aplicación y estados del sistema. |
| Acceso | Nginx, firewall Fortinet y router ISP | Encauzar el acceso web a los servicios de la instalación. |
| Despliegue | Docker sobre Raspberry Pi y Orange Pi 5 | Alojar los servicios de la plataforma en infraestructura de borde. |

## Tecnologías

`ESP32` · `C++` · `Node-RED` · `MQTT` · `Eclipse Mosquitto` · `MySQL` · `Nginx` · `Docker` · `Raspberry Pi` · `Orange Pi 5`

## Evidencias del proyecto

### Infraestructura de borde

![Servidores Raspberry Pi y Orange Pi 5 conectados al router ISP y al firewall Fortinet](docs/images/infraestructura-servidores.jpg)

### Flujo de automatización en Node-RED

![Flujo de Node-RED con controles manuales, riego automático temporizado y nodos MQTT](docs/images/flujo-nodered.png)

### Dispositivos de campo

| Caudalímetro | Electroválvula | Controlador y alimentación |
| --- | --- | --- |
| ![Caudalímetro instalado en la línea de agua](docs/images/caudalimetro.jpg) | ![Electroválvula montada en la tubería de riego](docs/images/electrovalvula.jpg) | ![Electrónica de control y fuentes de alimentación](docs/images/controlador-y-alimentacion.jpg) |

> El esquema refleja los componentes descritos para el proyecto. No se especifican aquí el reparto concreto de contenedores entre placas, la configuración de red, los tópicos MQTT ni las métricas de ahorro de agua.
