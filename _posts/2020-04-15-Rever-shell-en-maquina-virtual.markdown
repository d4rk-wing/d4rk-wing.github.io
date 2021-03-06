---
layout: post
title:  "Reverse Shell en máquina vírtual"
date:   2020-04-15 01:11:00 +0100
categories: Hacking
---

# Reverse Shell en máquina vírtual

## El problema
<br />Cuando se realizan ataques de explotación de vulnerabilidades que permiten establecer una sesión sobre la máquina victima, normalmente se utilizan payload los cuales nos brindan una Shell de línea de comandos de la victima sobre nuestra máquina atacante.
<br />Algunos profesionales de la seguridad suelen realizar este tipo de ataques desde una máquina vírtual con las herramientas necesarias para ello (Kali, Parrot, BlackArch, etc). Esta máquina vírtual suelen ejecutarla desde un equipo con sistema operativo Microsoft Windows.
<br />Dependiendo de los permisos que se tengan en la organización a la cual se le esta realizando la auditoria, un atacante podría configurar la tarjeta de red vírtual en modo puente/Bridge, lo cual permitiría tomar direccionamiento propio de la compañía y facilitar el tipo de ataques.
<br />Sin embargo, en la mayoría de las compañías, se tiene direccionamiento IP estático o reserva DCHP desde la MAC del equipo, y adicionalmente solo proporcionan una dirección IP desde la cual se ejecutan las pruebas de seguridad. Personalmente recomiendo tener el equipo con dual boot, para poder realizar las pruebas de seguridad directamente desde un sistema operativo Linux, aunque esto en algunas ocasiones no puede realizarse por limitaciones propias del cliente, en las cuales únicamente permite realizar conexiones desde sistemas operativos Windows a través de NAC u otros controles de seguridad.
<br />Retomando los ataques de sesión remota, al ejecutar el exploit (manual o automático) normalmente se recibe la sesión en el sistema operativo desde el cual se realiza la explotación. Por ejemplo desde Metasploit Framework, al ejecutar este tipo de ataques se establece una sesión de meterpreter o una reverse_tcp configurando la IP de la máquina atacante en la opción LHOST y especificando el puerto en LPORT. Lo anterior configura automáticamente el puerto en escucha para establecer la sesión.
<br />Pero al realizar este tipo de ataques desde una máquina vírtual con configuración de red NAT, la máquina vírtual adquiere una dirección IP privada la cual no es visible desde la máquina victima, por lo que al configurar esta dirección como "Local host" en el exploit, no se establece la sesión debido a que no es visible para la victima.

<br /><img src="/images/post/Reverse-shell-en-maquina-virtual/fail_exploit.png" width="100%" height="100%" />

## Posibles soluciones
<br />Al tener este problema, realice una busqueda en internet para poder dar una solución a la conexión en este tipo de escenarios. Al no encontrar una solución publicada en internet, pense en el tipo de solución que yo le daría a este problema. De esta manera se me ocurrieron dos posibles soluciones:
<br />
* Configurar un port forwarding / re-dirección de puerto local desde la máquina host a la máquina vírtual.
* Abrir un puerto de escucha desde el sistema operativo HOST Windows para recibir la sesión remota.
<br /><br />Para hacer un port forwarding que reciba la sesión y la envie a la máquina vírtual requiere realizar la configuración en el sistema operativo host y poner el puerto en escucha en la máquina vírtual para recibir la sesión desde el sistema operativo HOST. Sin embargo, se requieren menos configuraciones al simplemente poner a escuchar el puerto en la máquina host, por lo que opté por esta última dado que me pareció más práctica.
### Configuraciones previas a la explotación
<br />Para poder recibir la sesión remota, es importante que la victima tenga visibilidad del equipo desde el cual se realiza el ataque, por lo que se debe desactivar temporalmente el firewall de Windows y del Software antivirus (en caso de tener instalado) con el fin de garantizar la visibilidad.
<br /><img src="/images/post/Reverse-shell-en-maquina-virtual/firewall_host.png" width="100%" height="100%" />
<br />Adicionalmente, para recibir la sesión se debe hacer uso de un software que permita esta acción. En mi caso utilice Netcat para Windows el cual descargue del este [sitio](https://eternallybored.org/misc/netcat/){:target="_blank"}, aunque existen varios otros sitios e incluso repositorios de GitHub en los cuales puede encontrarse.
<br />Para este ejemplo, explotaremos la famosa vulnerabilidad de Eternalblue a través de Metasploit, también conocida por el boletín de seguridad correspondiente de Microsoft MS17-010 la cual fue utilizada en el ataque de ransomware WannaCry.
Para este escenario se tienen las siguientes máquinas:
1. Windows 10: Sistema operativo Host  
		Dirección IP: 192.168.0.7  
		<img src="/images/post/Reverse-shell-en-maquina-virtual/ipconfig_host.png" width="100%" height="100%" />
2. Windows 7: máquina vitual en vírtual Box - Victima  
		Configuración de red NAT/direccionamiento de la red local.  
		Dirección IP: 192.168.0.9  
		<img src="/images/post/Reverse-shell-en-maquina-virtual/ipconfig_victima.png" width="100%" height="100%" />
3. Kali Linux: máquina vírtual en VMware - Atacante  
		Configuración de red en Bridge/Visible desde el sistema operativo host.  
		Dirección IP: 192.168.17.129
		<img src="/images/post/Reverse-shell-en-maquina-virtual/ipconfig_atacante.png" width="100%" height="100%" />
<br />Se vírtualizan las dos máquinas en diferentes software de vírtualización con el fin de garantizar que no haya visibilidad al atacante desde la máquina victima.
### Ejecución del ataque
<br />___NOTA: En este post únicamente se explica cómo establecer la sesión remota en el sistema operativo host, por lo que no se detallan las configuraciones del exploit Eternalblue, este ataque se encuentra documentado en varios sitios de internet.___
<br />
1. Configurar puerto en escucha en Windows 10: *nc.exe -lvp [PUERTO]*
<img src="/images/post/Reverse-shell-en-maquina-virtual/host_escucha.png" />
2. Configurar IP victima en opción RHOST
3. En la opción LHOST se configura la dirección IP del sistema operativo anfitrión (Windows 10) y en LPORT el puerto anteriormente configurado.
4. Ejecutar el exploit (run o exploit)  
<img src="/images/post/Reverse-shell-en-maquina-virtual/eternal_config.png" width="100%" height="100%" />
<br />Como se puede observar se recibe correctamente la sesión en el sistema operativo HOST.  
<br /><img src="/images/post/Reverse-shell-en-maquina-virtual/ok_exploit.png" width="100%" height="100%" />
<img src="/images/post/Reverse-shell-en-maquina-virtual/shell_reverse.png" width="100%" height="100%" />
### Conclusiones
<br />Se ha podido establecer correctamente la sesión de un ataque de tipo Shell reversa realizado desde una máquina vírtual la cual no es visible desde la victima. Este método únicamente requiere la ejecución de Netcat para Windows y la desactivación de firewall en el sistema operativo, de esta manera se puede recibir la sesión remota en cualquier ataque de este tipo.
<br />Si bien al principio del post indiqué que no encontré ninguna publicación en internet que explicará este proceso, también es cierto que es un proceso simple que seguramente es usado por muchos profesionales que acostumbran realizar este tipo de pruebas desde una máquina vírtual. Sin embargo, no esta documentada para quienes estén iniciando en el mundo de las pruebas de ethical hacking
