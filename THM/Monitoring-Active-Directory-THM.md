# Monitoring Active Directory

https://tryhackme.com/room/monitoringactivedirectory

## Resumen de protocolos

| Protocolo | Puerto | Que hace | Usos |
|-----------|--------|----------|------|
| Kerberos | 88 | Autenticación en active directory | Inicios de sesión de usuarios, acceso a servicios y solicitudes de tickets |
| LDAP | 389, 636, 3268, 3269 | Consultas y modificaciones al directorio | Búsquedas de usuarios, verificación de suscripciones en grupos y consultas de libreta de direcciones |
| SMB | 445 (139 para NetBIOS legacy) | Compartición de archivos y administración remota | Acceso a carpetas compartidas, impresoras y diversas herramientas administrativas |
| RDP | 3389 | Acceso remoto a escritorio | Soporte técnico y administración de servidores |
| Name Resolution (Legacy) | 137, 138, 5355 | NetBIOS y LLMNR | Mecanismo de respaldo cuando DNS falle, usado por aplicaciones antiguas |

## Usuarios locale y de dominio

- Usuarios de dominio
Se autentican contra el controlador de dominio, almacenando sus credenciales de acceso en una base de datos (NTDS.dit). Cuando inician sesión, se forman eventos solo en el controlador de dominio(Domain controller). Esto permite ver comportamientos de manera centralizada en caso de que logee en  varias estaciones de trabajo
- Usuarios locales
Se autentican en contra de la base de datos SAM(Security Account Manager) de cada dispositivo. Los eventos se generarán solo en la máquina, formando parte de los logs internos de dicha máquina. 

Si se investiga eventos en una red, lo más claro es observar los eventos del controlador de dominio de manera centralizada

### Autenticación Kerberos
Kerberos es un protocolo de autenticación  que usa “tickets” usado por el Active directory para identificar sin usar contraseñas en la red. Cuando un usuario se autentica, primero solicita un TGT(Ticket Granting Ticket) al controlador de dominio. Dicho ticket se usa para solicitar tickets de servicio(TGS) para diferentes recursos. Los pasos son:
1. Solicitud TGT al controlador de dominio
2. Respuesta del controlador de dominio del tht
3. Solicitud TGS al controlador de dominio
4. Respuesta TGS del controlador de dominio
5. Presentar el ticket del servicio solicitado en el proceso al servidor objetivo

El event id de dichas autenticaciones son:
- Solicitud/respuesta TGT: 4768
- Solicitud/respuesta TGS: 4769
- Presentar ticket al servicio: 4624
En caso de que el usuario falle la autenticación(Fuerza bruta o que se haya equivocado de carácter), generará un evento con ID 4771

#### Tipos de encriptaciones en eventos 4768 y 4769

Las encriptaciones de seguridad que usan dichos tickets son:
- Encriptación RC4: Usada en sistemas más antiguos que no soportan AES. En sistemas de visualización de eventos, no pone directamente encriptación RC4, sino que pone 0x12
- AES-256: Usado para sistemas actuales(Del windows 2008 hacia arriba). Al igual que la encriptación RC4, no pone que es una encriptacion AES-256, sino que pone 0x17

Cabe recalcar que para ver dichos logs o eventos, se debe de usar un SIEM, por ejemplo Splunk, que nos permite ver características como ipv4s, tipos de encriptaciones, fecha y hora a las que sucedieron, dispositivos emisores y receptores,...

### Autenticación NTLM
Se usa NTML cuando kerberos no está disponible, lo cual sucede cuando:
- Se accede a recursos por la IP
- Cuando no se encuentra un sistema de destino en DNS 
- Cuando se intenta uno autenticar en sistemas que no son de domino

El proceso es más “simple” que kerberos:
1. El usuario se intenta conectar al servidor objetivo
2. El servidor pide al domain controller que valide las credenciales de acceso(El id del evento es 4776)
3. El domain controller devuelve un resultado de confirmación
4. El usuario se puede conectar al servidor creando una sesión(El id de proceso es 4624)

## Id de los eventos en base a contexto
Cada acción en una cuenta también tiene ids de eventos que se atribuyen a diferentes actividades de la misma:
### Eventos de cuentas
- Creación de la cuenta: 4720
- Permitir a la cuenta: 4722
- Intento de reseteo de la contraseña: 4724
- Deshabilitar cuenta:4725
- Cuenta bloqueada: 4740
### Eventos de grupos(membresías/suscripciones)
- Nuevo miembro añadido al grupo de seguridad global(todo el dominio): 4728
- Nuevo miembro añadido al grupo local de seguridad(Solo a nivel local de la máquina): 4732
- Miembro añadido al grupo universal de seguridad(Todo el “bosque”): 4756

### Eventos de servicios de los directorios
Se corresponden a los logs de eventos 5136, que se atribuye a modificaciones a nivel de atributos de características del active directory, es decir, el 5136 muestra el atributo cambiado y menciona específicamente que se ha cambiado.
- userAccountControl: Cambios en el estado de la cuenta
- servicePrincipalName: Modificaciones SPN
- scriptPath: Scripts que se ejecutan cuando el usuario logea
- member: Modificaciones a los niveles de atributo de grupos de suscripciones
- displayName, description, title 

### Seguimiento de modificaciones GPO(Group policy objects)
Los GPOs permiten a los administradores gestionar configuraciones en todo el dominio de manera centralizada(Uno solo puede configurar ajustes de seguridad, desplegar software,... en muchos dispositivos a la vez), por eso mismo los gpos son objetivos para los atacantes ya que si despliegan un software malicioso ahí, pueden desplegarlo en otras máquinas
Desde el SIEM adecuado, se pueden ver características de los cambios llevados a cabo en el GPO:
- Quien lo modifica
- Que gpo fue afectado
- Qué ha cambiado

Las políticas de seguridad del gpo no se verán reflejadas , por lo que si se hace algún cambio en laguna, el id de versión del gpo subirá, pero no se sabrá que cambio se ha hecho

### Eventos de inicios de sesión
Siempre que alguien inicia sesión en algún sistema, se almacena si la autenticación ha fallado o no, en eventos. Dichos eventos engloban a:
- Logins a un workstation
- Conexiones por red para compartir archivos
- Sesiones RDP

Los eventos con id de los mismos son:
- Login exitoso: 4624
- Login fallido: 4625

Luego, hay tipos de login(LogonType) que significan diferentes cosas, que pueden ayudar a entender lo que se estaba llevando a cabo y el contexto:
- Tipo 2: Interactivo(Uso de teclados o consolas)
- Tipo 3: Red(Acceso al compartir archivos, administración remota,...)
- Tipo 4: Lote(Tareas programadas que se ejecutan en las cuentas de usuarios)
- Tipo 5: Servicio(Servicios específicos que se ejecutan enn las cuentas de dicho servicio)
- Tipo 7: Desbloqueo(Usuario que activa/desbloquea un workstation)
- Tipo 10: Interacciones remotas(Sesiones de protocolo de escritorio remoto)

## Volumenes normales/anormales(Detección de comportamiento anómalo)
Un solo usuario puede loguear en un Active directory ya genera numerosos logs de diferentes sucesos/eventos, por lo que se suelen generar muchos en base a la cantidad de usuarios
- 50000-100000 de volumen diario de solicitudes de tickets(TGT): 4769
- 5000-10000 de solicitudes de tickets(TGT): 4768
- Logons: 4624

### Cuentas de ordenadores
Una gran parte del tráfico de Kerberos viene no de las cuentas de usuario , sino de las de los ordenadores. Para diferenciarlas, las cuentas de ordenadores terminan con un **$**. Dichas cuentas autentican de manera constante diferentes procesos como comunicación entre máquinas, actualizaciones, procesos automáticos,...llegando a generar mas o menos el 75% del tráfico en kerberos si hay saturación en el entorno

### Nombres de servicios comunes + patrones
Son servicios recurrentes que se relacionan con el event id 4769:
- krbtgt: Solicitudes de renovación TGT
- cifs: Acceso al compartir archivos
- ldap: Consultas al directorio
- http: Web application access
- MSSQLSvc: Acceso al servidor SQL
- HOST: Servicios generales del host

### Patrones de cuentas de servicio
Las cuentas de servicio son predecibles , ya que cuentan con las mismas fuentes, destinos, horas de comienzo,...
Tipos de cuentas:
- Cuentas de servicios SQL: Dan acceso a servidores con bases de datos desde servidores de aplicaciones
- Cuentas de servicios de backup: Da acceso a recursos compartidos de archivos desde servidores de backup
- Cuentas de monitorización: Sirven para consultar diversos sistemas
- Cuentas admin: Dan acceso a los workstations de los admin durantes las horas laborales

### Técnica de conteo de stack
La técnica de conteo de stack (Stack counting/long-tail analysis)es de las más útiles para detectar comportamiento anómalo, simplemente contando el numero de veces que cada valor aparece, dividiendo a los resultados en base  ala frecuencia de los mismos, y después analizando los mas extraños. Dicha técnica se puede aplicar a cualquier campo, filtrando por:
- Evento/os
- Account_name
- Client_Adress
- Service_Name
- Ticket_Encryption_Type

### Patrones basados en el tiempo
También, para evaluar si un comportamiento es anómalo o no, hay que tener en cuenta la hora a la que sucede un evento, es más raro que un evento pase a las 3 de la mañana que uno que ocurra en horario laboral(Esto no invalida ninguno de los dos como posible ataque)
- Logins de usuario: Normalmente suceden dentor del horario laboral
- Actividad de la cuenta del backup: Por automatización ,suele tomar lugar fuera del horario lectivo(Noche)
- Cuentas de trabajos por lotes: Normalmente organizadas por windows
- Uso de la cuenta de admin: Se espera que se use durante mantenimiento

## Configuración avanzada de la política de auditoría
La configuración avanzada de la política de auditoría se encuentra siguiendo esta ruta en el pc(Windows): Configuración→Políticas→Ajustes de windows→ Ajustes de seguridad→configuración avanzada de la política de auditoría(O similar) → Políticas de auditorías
Configuraciones mínimas para visibilidad del active directory:
- Login de cuenta(Autenticación de credenciales): 4776
- Login de cuenta(Servicio de autenticación de kerberos): 4768,4771
- Login de cuenta(Operaciones de servicios de tickets de kerberos): 4769
- Gestión de cuenta(Gestión de la cuenta del usuario): 4720,4722,4724,4725
- Gestión de cuenta(Gestión del grupo/os de seguridad): 4728,4732,4756
- Acceso a DS(Cambios en el servicio del directorio): 5136
- Logins/Logs out: 4624,4625
- Acceso a objetos(Compartición de archivos): 5140
