# Explotación de la vulnerabilidad de escalación de privilegios “Dirty Pipe”

Identificada con CVE 2022-0847. Permite sobrescribir el contenido de cualquier archivo en el caché de página, incluso si no es permitido que el archivo sea escrito, o si éste es inmutable; causando de que sea posible que un atacante malicioso sin privilegios en el equipo pueda realizar la técnica de explotación conocida como “elevación de privilegios” y tome posesión de este al permitirle abrir una terminal de usuario “*root*”. Principalmente debido a que el kernel siempre puede escribir en el caché de página, haciendo que escribir datos a un canal nunca requiera de ningún permiso.

![image](https://i.imgur.com/fyJ8hG3.png)

# Explotación de la vulnerabilidad

Primero se hablará de los prerequisitos a la explotación y luego se procederá a detallar los pasos a seguir para la explotación.

## Prerequisitos
1. Contar con Linux y una  versión del kernel distinta a:  5.16.11, 5.15.25 y 5.10.102.
> Verificar con:
> 
> uname -r
2. Actualizar todas las dependencias y paquetes del OS
> sudo apt update 
> 
> sudo apt upgrade
3. Descargar el Ubuntu Kernel Mainline  Installer e instalar una versión del kernel vulnerable.
> sudo add-apt-repository ppa:cappelikan/ppa
> 
> sudo apt install mainline
4. Ingresar a Advanced  Settings en el boot  menu.
5. Seleccionar la versión del kernel descargada.
6. Descargar el exploit de Github e instalar la librería gcc.
>sudo apt install gcc

## Explotación de la vulnerabilidad
1. Clonar el repositorio de Github que contiene el exploit:
> https://github.com/AlexisAhmed/CVE-2022-0847-DirtyPipe-Exploits
2. Ejecutar el compilador del exploit:
> chmod +x compile.sh
> 
> ./compile.sh
3. Ejecutar el exploit compilado:
> ./exploit-1

El resultado debe ser el siguiente:
![image](https://i.imgur.com/vc650oe.png)

4. Comprobar la ejecución del exploit:
> id
> whoami

![image](https://i.imgur.com/UrhDZQj.png)
#### De lo presentado anteriormente se puede apreciar que el escalamiento de privilegios fue exitoso, permitiéndole a un usuario sin privilegios, hacer uso de esta vulnerabilidad para cambiar la contraseña del usuario “_root_” y logrando obtener una consola con estos privilegios.
