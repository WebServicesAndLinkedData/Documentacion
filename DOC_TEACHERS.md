---
title: WSDL 18/19 - Teachers' Documentation 
---

## General structure of each repository
All the repositories will have a directory build where the files involved in testing the assignments will be kept. In specific assignments there can be separated files used for testing because of requeriments of other tools (example: maven in the third assignment)

## About the continuous integration service chosen: Semaphore
[Semaphore](https://semaphoreci.com/) has been chosen as the tool used to keep continuous integrated the repositories. To get started, login in their web with your github credentials and grant the permissions it requires. These permissions will be used to send mail notifications which can be disabled from the settings menu. On your first login in the service a windows will be shown to grant access to the services to all repositories you want to test. You can select them now, or add them manually later.

## Creación de builds para un repositorio en Semaphore
Para comenzar a realizar Integración continua sobre un repositorio, seleccionar en la parte superior de la web el botón de *Create New > Project*. Desde esa pantalla añadir el repositorio que se quiera en la sección correspondiente de la organización. En caso de que la organización no aparezca deberemos darle permisos, para ello, acudiremos a GitHub y accederemos en nuestro perfil *Settings > Applications > Authorized OAuth Apps > Semaphore* y en la parte inferior veremos todas las organizaciones a las se pertenece y daremos acceso con el botón *Grant* correspondiente de la misma.

Una vez hecho esto, quizás tengamos que esperar unos minutos para que los cambios se vean reflejados en la web de semaphore, pero cuando lo esten, seleccionar el respositorio que se quiere integrar, asi como la rama principal de prueba y el propietario del proyecto.
Después de esto, un escaneo automático comenzará para detectar el lenguaje por defecto, que podemos omitir pulsando *Skip the analysis* e indicarlo manualmente en la siguiente pantalla.
En esta pantalla realizaremos la configuración de prueba de la build. Se divide en tres secciones principales:

### Setup

Instrucciones a ejecutar antes de realizar las tareas de prueba. Dado que estan tareas van a ser ejecutadas desde un script debemos darles accesos de ejecución con la instrucción:
```bash
chmod +x ficheroTest.sh
```
Haremos esto por cada fichero bash que vayamos a ejecutar y tengamos en la carpeta build del repositorio.
Además crearemos el fichero donde escribiremos los mensajes de error para más tarde leerlo y formar el comentario:
```bash
touch err
```

### Jobs
Mandatos que se ejecutaran para comprobar si la build es válida o no. Principalmente llamara al script principal del directorio build que contendra todos las instrucciones necesarias. Esto se hara con la instrucción:
```bash
./build/ficheroTest.sh > err
```
Redireccionamos la salida al fichero err para escribir los mensajes de error y no obstaculizar la vista de los logs con estos mensajes.

Existe la posibilidad de de crear Jobs paralelos que ejecuten tareas de forma simultanea, teniendo en cuenta que deben ser independientes ya que se ejecutan en entornos separados.

### After Job
Por último configuramos las instrucciones a ejecutar al terminar la build. Principalmente se llamara al fichero que realiza el envio de comentarios:
```bash
./build/comment.sh
```

## Variables de entornos utilizadas
Para el correcto funcionamiento de muchos de los archivos de prueba es necesario incluir información delicada como Tokens de autorización API y demás. Para ello Semaphore cuenta con un sistema de variables de entorno que podemos acceder desde la sección *Project Settings > Environment Variables* de cada uno de los proyectos de Integración Continua. Allí pulsaremos *+ Add* y añadiremos la variable **TOKEN** que tendra como valor la clave API del usuario a relizar los comentarios. Por último marcaremos la casilla *Encrypt content* para reforzar la seguridad de la misma. 

## Consulta de resultados de las builds
En el panel de control del proyecto en Semaphore se puede consultar el estado de todas las builds realizadas haciendo click en la rama que se quiera observar o de la Pull Request a revisar. Haciendo click en el estado de una build podremos la salida por pantalla que ha producido cada mandato de ejecución.

# Descripción de los tests de cada tarea de la asignatura
Aquí se describiran los ficheros de prueba de cada una de las tareas de una forma similar a pseudocodigo.

#### comment.sh
Este fichero es común a todas las tareas por lo que será descritó aqui de forma común
1. Comprueba si el fichero *err* existe y no está vacio
    1. Si existe se parsea su contenido con el mandato *sed* para reemplazar carácteres invalidos para el Json
    2. Si no existe se establece un mensaje de exito por defecto
2. Se realiza la petición API para enviar el comentario

## Tests tarea 1
En esta tarea se ejecuta un script simple que evalua el fichero CSV.
#### testCSV.sh
1. Inicializa la variable de error a 0
2. Obtiene mediante una petición API a GitHub los datos de la Pull Request que son parseados con jq y se obtiene el nombre de usuario
3. Se comprueba si existe el fichero NombreDeUsuario.csv
    1. Si no existe muestra un mensaje de error y sale
    2. Si existe comprueba si todas las lineas tienen dos campos
        1. Si no se cumple se muestra un mensaje de error y sale
        2. Si se cumple no hace nada
4. Termina y sale con estado el número de errores encontrados

## Tests tarea 2
En esta tarea para realizar los tests, se ejecuta un *jar* pre-compilado por parte del profesor y se analizan los ficheros entregados por el alumno. Este *jar* se encuentra en la carpeta test.

#### testAssignment2.sh
1. Inicializa variables de error a 0 y falso
2. Obtiene mediante una petición API a GitHub los datos de la Pull Request que son parseados con jq y se obtiene el nombre de usuario
3. Se comprueba si existe el fichero NombreDeUsuario.rdf
    1. Si no existe muestra un mensaje y activa una variable de error
4. Se comprueba si existe el fichero NombreDeUsuario.ttl
    1. Si no existe muestra un mensaje y activa una variable de error
5. Se comprueba si existe el fichero NombreDeUsuario.png
    1. Si no existe muestra un mensaje de error
6. Si los dos ficheros rdf y ttl existen se realizan los tests ejecutando un fichero jar
    1. El estado de salida del mandato de ejecución de los tests se añade a la variable de error
7. Sale con el estado de error acumulado

## Tests tarea 3
En esta tarea se compila el codigo del alumno usando maven y se proceden a ejecutar los tests.

#### compileTasks.sh
1. Inicializa variables de error a 0
2. Obtiene mediante una petición API a GitHub los datos de la Pull Request que son parseados con jq y se obtiene el nombre de usuario
3. Se comprueba si existe el directorio NombreDeUsuario-NumMatricula
    1. Si no existe muestra un mensaje y sale
    2. Si existe copia todos los ficheros del alumno al directorio src y compila con maven
        1. Si hay problemas en la compilación muestra mensaje y sale
        2. Si no hay errores de compilación se procede a ejecutar los tests
            1. Si los tests no termina correctamente se imprime mensaje de error
4. El programa termina y sale con el estado de error