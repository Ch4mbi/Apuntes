# Network Discovery Detection

https://tryhackme.com/room/networkdiscoverydetection

Los atacantes quieren descubrir diferentes aspectos de una red a la que atacan ,normalmente siendo la de una compañía/empresa.Quieren averiguar datos como las IPs, OS, vulnerabilidades,...
Los defensores suelen usar diferentes programas para descubrir actividad en la red.
## Tipos de escaneos
### Escaneo de actividad externa
Los analistas SOC pueden detectar actividad anormal fuera de la red y escanear las maquinas dentro de la red que tengan acceso directo al exterior de la misma. Monitorean la ip fuente es externa y que la ip destino de las mismas en un dispositivo de la red interna. Si los atacantes llevaba  cabo un reconocimiento de la red, no se considera ataque, solo que están, posiblemente , identificando puntos de acceso a la red o vulnerabilidades. Los pasos para estos procesos de ataque suelen ser:
1. **Reconocimiento**
2. Desarrollo de recursos
3.Acceso inicial
4.Ejecución
5. Persistencia
6.Escalado de privilegios
7.Evasión de defensas
8.Acceso a credenciales
9.Descubrimiento
10.Movimiento lateral
11. Recolección
12 Exfiltración
13.C2
14. Impacto

El reconocimiento suele ser la primera fase de ataques y en esta fase, no tienen acceso como tal. Pero los analistas SOC pueden bloquear dicha ip antes de que actúe con el firewall.

### Escaneo de actividad interna
Los analistas soc también analizan las comunicaciones internas de la red(Entre dispositivos de la misma) con el objetivo de ver si hay comportamiento anómalo de un atacante que está en la fase de descubrimiento por ejemplo(Se le puede llamar reconocimiento interno desde el punto de vista de los atacantes)
1. Reconocimiento
2. Desarrollo de recursos
3.Acceso inicial
4.Ejecución
5. Persistencia
6.Escalado de privilegios
7.Evasión de defensas
8.Acceso a credenciales
9.**Descubrimiento**
10.Movimiento lateral
11. Recolección
12 Exfiltración
13.C2
14. Impacto

Este escaneo interno es mas importante que el externo ya que conlleva que un atacante ha obtenido acceso al interior de la red

### Escaneo horizontal
Un atacante puede escanear en varias ip un mismo puerto en específico, llamandose ese escaneo escaneo horizontal, el cual se usa para exponer un puerto en específico de varios hosts.
Esta clase de escaneos se puede detectar en base a las siguientes características:
1. Una única ip “externa”
2. Un solo puerto escaneado
3. Muchas ip de destino

### Escaneo vertical
El escaneo vertical ocurre cuando una sola ip es escaneada en múltiples puertos. Los atacantes hacen esto para detectar algún host e identificar sus puertos expuestos, permitiéndoles identificar vulnerabilidades en una sola máquina (y que la consideren un objetivo viable en base a sus objetivos).
Se puede detectar este escaneo en base a estas características:
1. Si los logs de escaneos vienen de la misma ip
2. La misma ip de destino se repite con la misma ip emisora
3.Varios ports de destino a lo largo de varios eventos

## Mecánicas de escáneres de red
### Ping sweep(Barrido de ping)
Es una de las técnicas más básicas de escaneo de red, normalmente usada para identificar host de una red. Funciona mandando un ICMP(Internet Control Message Protocol) al host. Pero es poco efectivo ya que existen medidas de seguridad que lo bloquean.

### Escaneos TCP SYN
Una conexión TCP se establece empezando por el 3-ways handshake para establecer conexión:
1. SYN
2.SYN-ACK
3. ACK
Los atacantes pueden usar este “saludo” para identificar hosts activos en la red y sus puertos abiertos. El escáner manda una solicitud SYN al objetivo. Si se recibe una respuesta SYN-ACK significa que el usuario víctima está activo/en línea. Es un escaner”sigiloso” ya que se junta con el resto de tráfico de la red

### Escaneo UDP
Este escaneo consiste en enviar un paquete UDP vacío. Si el puerto está cerrado, el host envia una respuesta ICMP de que el puerto no es alcanzable(es decir, que está cerrado), pero que el host si que está en linea. Aun así , en algunos casos, el escaner no recibirá ninguna respuesta hasta pasado un rato, significando una respuesta por temporizador y que el escaner marcará al puerto como abierto, pero no necesariamente lo estará. En caso de que el puerto esté abierto, el escaner recibirá un paquete UDP.
Escáneres UDP no son del todo fiables y son lentos, dependiendo de una respuesta.

===================================================================

Muchas organizaciones llevan a cabo escáneres internos para buscar vulnerabilidades o identificar superficie de ataque.
