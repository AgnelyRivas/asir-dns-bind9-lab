# asir-dns-bind9-lab

Laboratorio DNS con BIND9 en Ubuntu Server: zonas directa e inversa, configuración de red y pruebas desde un cliente Ubuntu.
Configuración y verificación de un servidor DNS en una red virtual privada, como práctica del ciclo Administración de Sistemas Informáticos en Red (ASIR), módulo Servicios de Red e Internet (curso 2026–2027).

## Objetivo

Implantar un servicio de resolución de nombres para el dominio de laboratorio ejemplo.prueba, de modo que un cliente Ubuntu pueda consultar los registros DNS de los equipos de su red sin depender de un servicio DNS externo para ese dominio.

## Entorno y tecnologías

- Servidor: Ubuntu Server 26.04 LTS, sin escritorio gráfico.
- Cliente: Ubuntu Desktop.
- Virtualización: Oracle VirtualBox, red interna 172.16.0.0/24.
- DNS: BIND9; registros SOA, NS, A, CNAME y PTR.
- Configuración de red: Netplan, systemd-resolved y NetworkManager (nmcli).
- Verificación: named-checkconf, named-checkzone, host, nslookup, ping y systemctl.
  
## Topología del laboratorio

| Equipo | Función | Dirección IP |
|---|---|---|
| Ubuntu Server | Servidor DNS (`ns1.ejemplo.prueba`) | `172.16.0.1/24` |
| Ubuntu Desktop | Cliente (`pc1.ejemplo.prueba`) | `172.16.0.66/24` |

Ambos equipos utilizan una red interna de VirtualBox. El servidor dispone además de una interfaz NAT para conectividad externa durante la instalación.

## Configuración implementada

1. Instalación de BIND9 y sus utilidades.
2. Configuración de recursión, restricciones de consultas y reenviadores externos (8.8.8.8 y 8.8.4.4).
3. Creación de la zona directa ejemplo.prueba: registros de autoridad, servidor DNS, host pc1 y alias recepcion.
4. Creación de la zona inversa 0.16.172.in-addr.arpa: registros PTR para el servidor y el cliente.
5. Declaración de las zonas en /etc/bind/named.conf.local.
6. Configuración de BIND9 para IPv4, ajustes de resolución local y parámetros de red.
7. Configuración de la dirección IP y del DNS en el cliente Ubuntu.
8. Comprobaciones de sintaxis, servicio, conectividad y consultas DNS.
Adaptación al entorno: en esta instalación no estaban disponibles las plantillas /etc/bind/db.local y /etc/bind/db.empty. Por ello se creó el contenido de las zonas manualmente, respetando los registros previstos en la práctica. En el cliente se utilizó nmcli porque el panel gráfico de red provocaba bloqueos.

## Instalación y reproducción

Estos pasos están pensados solo para un laboratorio aislado. Antes de repetirlos, ajusta las interfaces, las IP y el nombre de dominio a tu red. No copies archivos de ejemplo directamente a un servidor en producción.
1. Crea dos máquinas virtuales Ubuntu, con sus interfaces conectadas a la misma red interna de VirtualBox.
2. Asigna al servidor 172.16.0.1/24 y al cliente 172.16.0.66/24 en sus interfaces de laboratorio.
3. En el servidor, instala los paquetes:
   
```bash
sudo apt-get update
sudo apt-get install bind9 bind9utils
```

4. Configura /etc/bind/named.conf.options limitando la recursión y las consultas a 127.0.0.1 y 172.16.0.0/24, con reenviadores externos. Mantén las transferencias deshabilitadas si no utilizas un segundo servidor.
5. Crea /etc/bind/db.ejemplo.prueba con los registros de la zona directa y /etc/bind/db.172.16.0 con los PTR de la zona inversa. Registra ambas zonas como type master en /etc/bind/named.conf.local.
6. Comprueba los archivos antes de reiniciar:
   
```bash
sudo named-checkconf /etc/bind/named.conf.options
sudo named-checkzone ejemplo.prueba /etc/bind/db.ejemplo.prueba
sudo named-checkzone 0.16.172.in-addr.arpa /etc/bind/db.172.16.0
sudo systemctl restart bind9
sudo systemctl status bind9
```
7. Configura el cliente con IP 172.16.0.66/24, DNS 172.16.0.1 y dominio de búsqueda ejemplo.prueba, mediante su gestor de red.
8. Comprueba conectividad y resolución:

```bash
ping -c 4 172.16.0.1
nslookup pc1.ejemplo.prueba 172.16.0.1
nslookup ns1.ejemplo.prueba
```

## Resultados observados

- El servicio BIND9 mostró estado active (running).
- Las zonas directa e inversa superaron la validación con named-checkzone.
- El cliente logró conectarse al servidor mediante ping, con 0 % de pérdida en la prueba documentada.
- La consulta pc1.ejemplo.prueba devolvió 172.16.0.66.
- La consulta ns1.ejemplo.prueba devolvió 172.16.0.1.

## Incidencia pendiente

La resolución del nombre corto ns1 devolvió SERVFAIL, aun teniendo ejemplo.prueba configurado en NetworkManager. Durante el diagnóstico, /etc/resolv.conf mostraba search ., mientras que resolvectl mostraba el dominio de búsqueda en la interfaz. No se documenta una corrección final verificada. Esta incidencia queda registrada para su estudio.

También se observaron avisos de resolución externa en los registros de BIND9, sin que impidieran las consultas locales verificadas.

## Evidencias y documentación

El informe académico original contiene capturas del proceso de instalación, configuración y pruebas. Para publicación pública, utiliza versiones seleccionadas y depuradas de las capturas, evitando exponer credenciales, identificadores personales y metadatos innecesarios.

Estructura recomendada del repositorio:

```bash
asir-dns-bind9-lab/
├── README.md
├── docs/
│   └── informe-publico.pdf
└── screenshots/
    ├── 01-bind9-servicio.png
    ├── 02-zona-directa.png
    ├── 03-zona-inversa.png
    ├── 04-zonas-declaradas.png
    ├── 05-cliente-red.png
    └── 06-consulta-dns.png
```

Los archivos de docs/ y screenshots/ son opcionales y deben incorporarse únicamente tras revisar y anonimizar las evidencias. La estructura anterior es una propuesta, no un listado de archivos ya publicados.

## Alcance y limitaciones

Este repositorio documenta un laboratorio educativo, no un despliegue en producción. La configuración incluía reenviadores y restricciones de acceso, pero no se implementaron ni verificaron transferencias entre servidores DNS, actualizaciones DNS dinámicas ni alta disponibilidad. Tampoco se considera completada la resolución por nombre corto.

## Aprendizajes

- Administración de un servicio de red desde la terminal Linux.
- Relación entre zonas DNS, registros y consultas de clientes.
- Diagnóstico por capas: conectividad IP, resolución de nombres y estado del servicio.
- Adaptación de instrucciones a diferencias reales entre versiones de Ubuntu y BIND9.
- Documentación de evidencias y limitaciones de un laboratorio técnico.

## Autoría

Práctica académica de ASIR, módulo Servicios de Red e Internet. El informe de clase identifica a su autora; este README está redactado para una publicación técnica que no requiere mostrar datos personales completos.
