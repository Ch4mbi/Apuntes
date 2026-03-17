Primero habría que hacer ipconfig para conocer la red en la que uno se encuetra

[https://nmap.org/man/es/man-briefoptions.html](https://nmap.org/man/es/man-briefoptions.html)

Si no funciona es posible que:
- Haya que hacer sudo su y ejecutar el comando desde ahí
- O poner sudo antes
## Comando general(Linux)

  
``` bash
usuario@linux~$: nmap “comandos” 192.168.1.1 (O la ip objetivo dependiendo del contexto[También se usa para descubrir usuarios de una misma red ppor ejemplo])
```
- Una red completa:
``` bash
usuario@linux~$: nmap 192.168.1.0/24
```
- Varias IP:
``` bash
usuario@linux~$:nmap 192.168.1.10 192.168.1.20 ...
```
- Escanear desde archivo
``` bash
usuario@linux~$:nmap -iL lista.txt
``` 
## Comandos

### Descubrimiento de hosts

- -sn

  Se usa para saber que hosts/equipos estan activos en una red sin escaneo de puertos
- - Pn
  
  Desactiva el descubrimiento de hosts, haciendo a nmap asumir que el host está activo, pasando al escaneo de puertos
- - PS
  
  Lleva a cabo un "descubrimiento automático" a puertos específicos(Siendo por defecto el 80). Es mejor escribir algo como -PS22,80
### Escaneos de puertos
- - sS
  
  Lleva a cabo un escaneo TCP SYN enviando paquetes SYN para identificar puertos sin completar la conexion
- - sT
  
  Realiza un escaneo TCP connect estableciendo asi le conexion completa para determinar si el puerto está abierto
- - sU
  
  Hace un escaneo de puertos UDP, enviandolos para identificar servicios que usen ese protocolo
- - sA
  
  Hace un escaneo ACK, para determinar si los puertos están filtrados por un firewall. No identifica puertos abiertos
- - sN
  
  Lleva a cabo unescaneo Null enviando paquetes TCP sin flags, lo cual sirve para detectar puertos abiertos o cerraods en sistemas que sigan el estandar TCP
- - sF
  
  Realiza un escaneo FIN enviando paquetes con el flag FIN activado. Se usa para evadir algunos sistemas de filtrado para detectar puertos
- - sX
  
  Realiza un escaneo Xmas enviando paquetes con los flags FIN,PSH y URG activados para identificar el estado de los puertos

#### Como escanear puertos
- - p 80
  
  Escanea el puerto especificado(80 - http)
- - p 22,80,443,...
  
  Escanea los puertos especificados(22 - TCP, 80 - HTTP ,443
- - p1-1000
  
  Escanea el rango de puertos determinado
- - p-
  
  Escanea todos los puertos
- - F
  
  Escanea los 100 puertos "más comunes"
- --top-ports 500

  Escanea los 500 puertos mas usados
-  -p- --open
  
  Solo muestra puertos abiertos

### Lo mas usado
#### Detección de versión y servicios
Es lo que mas usa la gente tras detectar puertos abiertos:
- - sV
  
Detecta versión del servicio
- - sV --version-intensity
  
Nivel de agresividad, va del 0 (minimo) - 9 (Maximo)
- - A
  
Escaneo agresivo el cual incluye  versión ,OS, y scripts
#### Detección del OS
- - O
  
Detección de OS
- --osscan-limit

Limita la busqueda de OS a solo los prometedores
- --osscan-guess

Intenta adivinar, pero no suele dar respuestas fiables
#### Escaneo de scripts NSE
- --script vuln

Ejecuta scripts relacionados con vulnerabilidades
- --script default

Ejecuta scripts por defecto
- -sC

Equivalente a script == default

#### Timing/Velocidad
- -T0

Extremadamente lento
- -T1

Muy lento
- -T2

Lento
- -T3

Normal, lo que normalmente se usa por defecto
- -T4

Relativamente rápido
- -T5

Muy rápido
- --min-rate 1000

Envia minimo 1000 paquetes(o los que se pongan)
- --max-retries 2

Reduce el número maximo de intentos
#### Salida de archivo
- -oN ejemplo.txt

Salida normal(txt)
- -oX

Salida en XML
- -oG

Salida grepeable
- -oA

Guarda en los 3 formatos a la vez
- -d

Modo debug
#### Escaneos completos de red
- Escaneo completo(Util para auditorias)

``` bash
usuario@linux~$:nmap -p- -sS -sV -sC -O -T4 192.168.1.1
```
- Escaneo completo agresivo
``` bash
usuario@linux~$:nmap -p- -A -T4 192.168.1.1
```
- Escaneo UDP+TCP
``` bash
usuario@linux~$:nmap -sS -sU -p- 192.168.1.1
```

#### Evasion de firewalls
- f

Fragmenta paquetes, lo cual es un metodo de "evasión" del firewall
- --mtu 8

Especifica el tamaño del/los fragmentos
- -S IP

Hace spoofing a la dirección ip de origen
- --source-port 53

Usa un puerto de origen especifico
- --data-length 200

Añade datos aleatorios
#### IPv6
- -6

Escaneo de IPv6
``` bash
usuario@linux~$:nmap -6 fe80::1
```

