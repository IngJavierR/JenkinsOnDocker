# Ejercicio CI/CD con Jenkins  

Levantar ambiente CI/CD con Jenkins, Sonarqube, Nexus

## Prerequisitos Docker

Para poder utilizar la instancia local de Nexus para la subida de las imágenes de docker es necesario decirle al engine que nuestro registry local que de momento responderá por http es seguro, para lo cual necesitamos ingresar al Docker dashboard, ingresar al engrane en la esquina superior derecha, accesar a la opción Docker Engine y agregar la sección:

```bash
 "insecure-registries": [
    "xx.xx.xx.xx:8083"
  ]
```

Donde las xx.xx.xx.xx serán remplazadas por la ip de la maquina donde ejecutamos este ejercicio.

![Docker Configuration](/assets/img/docker_configuration.png "Docker Configuration")

## Instalar Jenkins con Docker

Crear una carpeta llamada jenkins_home

```bash
mkdir jenkins_home
```  

Posicionados a nivel de esa carpeta (no adentro) ejecutar para el SO correspondiente:

```bash
Linux/Mac
docker run -d -p 8080:8080 -p 50000:50000 -v $(PWD)/jenkins_home:/var/jenkins_home -v /var/run/docker.sock:/var/run/docker.sock jenkins-local

Windows
docker run -d -p 8080:8080 -p 50000:50000 -v "[Path]/jenkins_home:/var/jenkins_home" -v "//var/run/docker.sock:/var/run/docker.sock" jenkins-local
```

Donde:  

* Path (solo aplica para windows) y debe tener la ruta absoluta (C:\Windows...)  

Una ves instalado accesar a la ruta: <http://localhost:8080/> que deberá mostrar la siguiente pantalla:

![Unlock Jenkins](/assets/img/unlock_jenkins.png "Unlock Jenkins")

Para obtener el Administrator password que nos solicita accesamos al log del contenedor de jenkins.  

```bash
docker ps
```  

![Container ID](/assets/img/get_containerid.png "Container ID")

Obtenemos el ContainerID y posterior ejecutamos:  

```bash
docker logs [ContainerID]
ej. docker logs cae27db9a1b0
```  

Nos mostrará el log con el password que deberemos de utilizar para desbloquear la instalación de Jenkins y damos click en "Continuar".  

![Jenkins Password](/assets/img/jenkins_password.png "Jenkins Password")  

Seleccionamos la opción de "Install Suggested Plugins".  

![Suggested Plugins](/assets/img/suggested_plugins.png "Suggested Plugins")  

Debemos de esperar a que la instalación de los plugins finalice.

![Installing Plugins](/assets/img/instaling_plugins.png "Installing Plugins")  

Al terminar capturar los datos solicitados para la creación del usuario administrador, si se trata de una instalación de prueba, capturar datos demo, no se realizará ningún tipo de validación de informacion. Para avanzar click en el botón "Save and Continue"

![Admin User](/assets/img/admin_user.png "Admin User")  

En la siguiente pantalla solo confirmamos la url de nuestra instancia de Jenkins y damos click en "Save and Finish".  

![Instance Configuration](/assets/img/instance_configuration.png "Instance Configuration")  

Si llegamos a esta pantalla quiere decir que hemos finalizado la instalación de forma correcta, presionar el botón "Start using Jenkins".  

![Start Jenkins](/assets/img/start_jenkins.png "Start Jenkins")  

## Instalar Sonarqube con Docker

Para instalar Sonarqube en docker hay que ejecutar el comando:

```bash
docker run -d --name sonarqube -e SONAR_ES_BOOTSTRAP_CHECKS_DISABLE=true -p 9000:9000 sonarqube:latest
```

Una ves instalado accesar a la ruta: <http://localhost:9000/> que deberá mostrar la siguiente pantalla:

![Login Sonarqube](/assets/img/login_sonarqube.png "Login Sonarqube")  

Accesar con las credenciales de:

* Usuario: admin
* Password: admin

Posteriormente hay que cambiar la contraseña del usuario administrador.

![Change Password](/assets/img/change_password.png "Change Password")  

Si podemos ver la siguiente pantalla quiere decir que la instalación a finalizado satisfactoriamente.

![Sonarqube Success](/assets/img/sonarqube_success.png "Sonarqube Success")  

## Instalar Nexus con Docker

Para instalar Nexus Sonatype en docker hay que ejecutar el comando:

```bash
docker run -d -p 8081:8081 -p 8082:8082 -p 8083:8083 --name nexus sonatype/nexus3
```  

Una ves instalado accesar a la ruta: <http://localhost:8081/> que deberá mostrar la siguiente pantalla:

![Nexus Installation](/assets/img/nexus_instalation.png "Nexus Installation")  

### Configurar repositorio de imágenes Docker en Nexus  

Necesitamos hacer login en Nexus en el boton de login en la esquina superior derecha, para lograrlo debemos de consultar el archivo "/nexus-data/admin.password" ubicado dentro den contenedor, entonces ejecutaremos el siguiente comando utilizando el Container ID de Nexus.  

```bash
docker exec -it [ContainerID] bash -c "cat /nexus-data/admin.password"
```  

El resultado del comando deberá ser el password temporal de administrador que ingresaremos en la pantalla de login.

![Nexus Login](/assets/img/nexus_login.png "Nexus Login")  

Nos aparecerá la pantalla de Setup donde habrá que seguir los pasos que aparecen con los datos solicitados.  

![Nexus Change Password](/assets/img/nexus_change_password.png "Nexus Change Password")  

![Nexus Disable Anonymous Access](/assets/img/disable_anonymous_access.png "Nexus Disable Anonymous Access")  

Ahora debemos de configurar 3 repositorios para la administración de las imágenes de Docker:

* Repositoro privado (Hosted) para las imágenes propias
* Repositorio tipo proxy que apunte a DockerHub
* Repositorio agrupador para unir ambos repositorios en una sola URL

Ingresar al icono del engrane en la barra de navegación y seleccionar la opción Repositorios.

![Nexus Configurar Repositorio](/assets/img/entrage_repositorios.png "Nexus Configurar Repositorio")  

#### Configurar repositoro privado (Hosted)  

Ir a la opción: "Create repository / docker (hosted)" y colocar las siguientes opciones, luego hacer click en "Create repository":

![Nexus Docker Hosted](/assets/img/nexus_docker_hosted.png "Nexus Docker Hosted")  

#### Configurar repositoro publico (Proxy)  

Ir a la opción: "Create repository / docker (proxy)" y colocar las siguientes opciones, luego hacer click en "Create repository":

![Nexus Docker Proxy](/assets/img/docker_nexus_proxy.png "Nexus Docker Proxy")  

#### Configurar grupo de repositorios (Group)  

Ir a la opción: "Create repository / docker (group)" y colocar las siguientes opciones, luego hacer click en "Create repository":

![Nexus Docker Group](/assets/img/docker_nexus_group.png "Nexus Docker Group")  

### Permitir la conexión con repositorios Docker que no se exponen por https  

Al ser un ejercicio donde no hemos habilitado SSL, debemos de decirle al Docker engine que nos permita conectarnos a este repositorio por lo que abriremos la consola de Docker Desktop.  

Posterior ya podríamos ejecutar el comando:

```bash
docker login -u admin -p admin123 [ip-local]:8082
```  

## Configurar Jenkins, Sonarqube y Nexus  

Ahora vamos a configurar el ecosistema de Jenkins para conectarse a los demás componentes.  

### Instalar plugins  

Desde la consola de Jenkins, accesar a al menú "Administrar Jenkins" y despues "Gestor de Plugins", buscar e instalar los plugins de:

* SonarQube Scanner
* OWASP Dependency-Check
* Gatling

![Jenkins Plugin](/assets/img/jenkins_plugins.png "Jenkins Plugin")  

Hacer check en los tres plugins y presionar el botón "Download now and install after restart".  

En la siguiente pantalla hacer scroll hasta abajo y seleccionar la casilla de "Reiniciar Jenkins cuando termine la instalación y no queden trabajos en ejecución".  

![Jenkins Restart](/assets/img/jenkins_restart.png "Jenkins Restart")  

Es normal que en algunos casos al hacer el reinicio en contenedor de Jenkins se detenga, por lo que hay que levantarlo nuevamente con los comandos:

```bash
docker ps -a
```

Obtener el ContainerID de Jenkins y ejecutar:

```bash
docker start [Container ID]
```  

Despues volver a entrar a Jenkins y hacer login, validar que los plugins esten instalados.  

### Configurar el cliente de Sonarqube en Jenkins  

Ingresar a sonarqube y hacer click en el icono de usuario en la esquina superior de la pantalla, seleccionar la opción: "My Account".

![Sonarqube MyAccount](/assets/img/sonarqube_myaccount.png "Sonarqube MyAccount")  

Ingresar a la opción "Security" y generar un nuevo token del tipo "User Token" que expire en 90 días (recordar renovarlo constantemente). Copiar y resguardar el token, ya que una vez que salgamos de la pantalla no será posible recuperarlo.

![Sonarqube Create Token](/assets/img/sonarqube_create_token.png "Sonarqube Create Token")  

Para dar de alta el token en Jenkins debemos de regresar a la consola e ingresar a la opción: "Administrar Jenkins / Manage Credentials" y seleccionar la opción "Add Credentials" como se muestra en la imagen.  

![Jenkins Add Credentials](/assets/img/jenkins_addcredentials.png "Jenkins Add Credentials")  

Seleccionar credencial del tipo "Secret text", en el campo "Secret" colocar el token de Sonarqube, en el campo "ID" el nombre con el que haremos referencia a este token y en "Description" un texto breve que haga referencia al uso de este token. Hacer click en el botón "Create".  

![Jenkins Add Secret](/assets/img/jenkins_add_secret.png "Jenkins Add Secret")  

Ya con el token almacenado ingresar al menú: "Administrar Jenkins / Configurar el Sistema" y hacer scroll hasta la sección SonarQube servers donde haremos click en el botón "Add SonarQube" y configuraremos los siguientes datos:  

* Name: SonarServer
* URL del servidor: Colocar la URL de sonarqube, en lugar de localhost ingresar la ip de nuestro equipo (se obtiene con ipconfig o ifconfig dependiendo el SO)
* Server authentication token: Seleccionar de la lista el token que acabamos de generar
* Hacer click en el botón de "Guardar"

![Set Sonarqube Server](/assets/img/set_sonarqube_server.png "Set Sonarqube Server")  

### Configurar Maven en Jenkins  

Ingresar al menú "Administrar Jenkins / Global Tool Configuration" hacer scroll hasta la sección Maven, dar click en "Añadir Maven" y colocar un nombre para la instalación y seleccionar una versión. Para finalizar damos click en el botón "Save".  

![Set Jenkins Maven](/assets/img/jenkins_maven.png "Set Jenkins Maven")  

### Configurar Dependency Check en Jenkins y Sonarqube

Ingresar al menú "Administrar Jenkins / Global Tool Configuration" hacer scroll hasta la sección Dependency-Check, dar click en "Añadir Dependency-Check" y colocar un nombre para la instalación y seleccionar una versión. Para finalizar damos click en el botón "Save".  

![Jenkins DependencyCheck Install](/assets/img/jenkins_dependencycheck_install.png "Jenkins DependencyCheck Install")  

Posterior a esto ingresamos a Sonarqube, entramos a al menú "Administration / Marketplace" y buscamos el plugin "Dependency-Check", para que se nos permita la instalación, debemos de hacer click en el botón "I understand the risk" que nos advirte sobre el uso de plugins de terceros.  

![Sonarqube DependencyCheck Install](/assets/img/sonarqube_dependencycheck_install.png "Sonarqube DependencyCheck Install")  

Una vez aceptado se habilita el botón de install y una vez terminado nos solicita un restart del server para continuar.  

![Sonarqube Restart](/assets/img/sonarqube_restart.png "Sonarqube Restart")  

