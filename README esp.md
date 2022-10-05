# Tutorial sobre creación de una aplicación de lago de datos (DataLake)

## Configuración de herramientas

Estamos utilizando la siguiente pila tecnológica:

- Apache NiFi para procesar y distribuir datos.
- Registro Apache NiFi para almacenar, administrar y controlar versiones de recursos NiFi.
- Apache Airflow para crear, programar y monitorear flujos de trabajo mediante programación.
- Postgres como base de datos objeto-relacional.
- pgAdmin como plataforma de administración y desarrollo de la base de datos postgres.
- MinIO como un almacenamiento de objetos compatible con S3 alojado localmente.

![images/diagram](images/diagram.png "Diagram")

Apache NiFi admite gráficos dirigidos potentes y escalables de enrutamiento de datos, transformación y lógica de mediación del sistema.

Apache NiFi Registry es un subproyecto de Apache NiFi y es una aplicación complementaria que proporciona una ubicación central para el almacenamiento y la gestión de recursos compartidos en una o más instancias de NiFi. Lo usaremos para controlar la versión de nuestros flujos de datos y para crear plantillas para uso repetido.

Apache Airflow es escalable debido a su arquitectura modular y utiliza colas de mensajes para orquestar cualquier número de trabajadores. Sus pipelines están escritas en Python, lo que significa que permite la creación dinámica de pipelines a partir del código y es extensible con operadores personalizados.

PostgreSQL es un poderoso sistema de base de datos relacional de objetos de código abierto que se ha ganado una sólida reputación por su confiabilidad, robustez de funciones y rendimiento.

pgAdmin como plataforma de administración y desarrollo de PostgreSQL.
pgAdmin es una plataforma de desarrollo y administración de bases de datos de código abierto para la base de datos PostgreSQL

MinIO como sustituto alojado localmente para AWS S3 como almacenamiento de objetos.
MinIO ofrece almacenamiento de objetos compatible con S3 de alto rendimiento.

### Parte 1: Requisitos previos

1. Instale docker: <https://www.docker.com/products/docker-desktop> (**Si usa Windows, deshabilite WSL durante la instalación, usaremos HyperV**)

![images/disablewsl.png](images/disablewsl.png "disablewsl")
1. git clone <https://github.com/sercasti/datalaketools.git>
1. (Si usa Windows) Habilite el uso compartido de archivos en la aplicación de escritorio Docker sobre la carpeta donde clonó este repositorio
![images/settingsDocker.png](images/settingsDocker.png "settingsDocker")
1. Usando su consola de línea de comandos (cmd o terminal), ingrese a la carpeta datalaketools y ejecute: docker-compose up

### Parte 2: conectar pgAdmin a la base de datos de postgres

Una vez que se ejecutan los servicios "docker-compose up", puede acceder a pgAdmin en su navegador: <http://localhost:5050/>, le pedirá que establezca una contraseña maestra. Elija una contraseña que pueda recordar.

  1. Haga clic en "Agregar nuevo servidor" (Add New Server) en el medio de la página debajo de "Enlaces rápidos" (Quick Links)
  1. En la pestaña General: elija un nombre para su servidor de base de datos, e.g. postgres_db
  1. En la pestaña Conexión: El "Nombre de host/dirección" use el nombre de host mypostgres.
  1. En la pestaña Conexión: El nombre de usuario y la contraseña se especifican en docker-compose.yml como variables de entorno del servicio postgres ("postgres" y "postgres" si aún no lo ha cambiado).

### Parte 3: Apache NiFi al registro de NiFi (y viceversa)

Acceda al registro de NiFi en su navegador: <http://localhost:18080/nifi-registry>

  1. Haga clic en el símbolo de llave inglesa en la esquina superior derecha de la ventana.
  1. Haga clic en "NUEVO CUBO" (NEW BUCKET) en el lado derecho.
  1. Ingrese un nombre para su cubo, por ejemplo, "miprimercubo" y luego use el botón "Crear".
  1. Dirígete a <http://localhost:8091/nifi/> y haz clic en las tres barras en la esquina superior derecha para acceder a "Configuración del controlador".
  1. A continuación, haga clic en la pestaña "Registrar clientes" y en el símbolo más del lado derecho.
  1. Ponga el nombre que desee (es decir, "nify-registry") y URL <http://myregistry:18080>

### Parte 4: Apache NiFi a la base de datos postgres

Acceda a NiFi a través de (<http://localhost:8091/nifi/>)

  1. Arrastre y suelte un "Grupo de procesos" (cuarto icono) en el espacio en blanco. Establezca un nombre y haga clic en "agregar"
  1. Haga clic derecho en el grupo de procesos y haga clic en "Configurar"
  1. En la pestaña "Servicios de controlador", agregue un nuevo controlador haciendo clic en el símbolo más en el lado derecho. Elija DBCPConnectionPool de la larga lista de controladores disponibles y haga clic en Agregar.

Edite las propiedades del controlador recién creado (usando el icono de engranaje) de la siguiente manera:

- URL de conexión de la base de datos: jdbc:postgresql://mypostgres:5432/postgres
- Nombre de la clase del controlador de la base de datos: org.postgresql.Driver
- Ubicaciones del controlador de la base de datos: /opt/nifi/nifi-current/jdbc/postgresql-42.3.3.jar
- Usuario de la base de datos: postgres (o lo que haya cambiado durante la configuración)
- Contraseña: postgres (o lo que sea que haya cambiado durante la configuración)

Haga clic en Aplicar y habilite el servicio seleccionando el símbolo del rayo de la línea que representa el controlador recién creado y eligiendo Habilitar en la ventana que se abre.

### Parte 5: Apache NiFi a MinIO

Acceda al servicio en su navegador en <http://localhost:9001/dashboard>. El nombre de usuario y la contraseña se configuran en el archivo docker-compose.yml a través de los parámetros de entorno MINIO_ROOT_USER y MINIO_ROOT_PASSWORD. A menos que los haya cambiado, son minio_admin y minio_pass
palabra.

  1. Haga clic en Cubos (Buckets) en la barra de navegación de la izquierda y haga clic en Crear cubo  (Bucket) en la esquina superior derecha.
  1. Elija un nombre para su depósito, e.g. miniobucket, mantenga las configuraciones predeterminadas y haga clic en Guardar
  1. Seleccione su depósito y haga clic en Examinar.
  1. Cree un nuevo directorio usando el botón "Crear nueva ruta", llámelo "prueba" y dentro de él haga clic en el símbolo Cargar archivo para cargar minio_testfile.txt que puede encontrar en la raíz del directorio clonado "datalaketools".
  
### Parte 6: Apache Airflow a la base de datos postgres

Abra el servicio Airflow en su navegador en <http://localhost:8085/admin/> y haga clic en Admin -> Conexiones en la barra superior. Airflow viene con muchas conexiones de forma predeterminada, pero creemos una nueva para nuestro propósito.

Haga clic en Crear y complete los detalles necesarios:

- Conn Id: mypostgres_connection: el ID con el que podemos recuperar los detalles de la conexión más adelante.
- Tipo de conexión: Postgres - Selecciónelo en el menú desplegable.
- Host: mypostgres - Docker resolverá el nombre de host.
- Esquema: postgres - el nombre de la base de datos (la etiqueta es engañosa)
- Inicio de sesión: postgres - o cualquier nombre de usuario que establezca en su archivo docker-compose.yml.
- Contraseña: postgres, o la contraseña que establezca en su archivo docker-compose.yml.
- Puerto: 5432: el puerto estándar para la base de datos dentro de la red docker.

Luego haga clic en "guardar"

1. Regrese a la página principal de Airflow, busque el DAG "hello_postgres_postgres_operator" y actívelo en la barra de herramientas izquierda, luego haga clic en "Activar DAG" en el lado derecho (primer icono).

### Parte 7: Apache Airflow a Apache NiFi

Dentro del servicio Airflow en su navegador en <http://localhost:8085/admin/>, creemos una conexión para nuestro servicio NiFi de la siguiente manera. Haga clic en Admin -> Conexiones en la barra superior. Haga clic en "Crear"

- ID de conexión: mynifi_connection
- Tipo de conexión: Postgres (dado que no hay un tipo de conexión NiFi, usamos postgres como sustituto).
- Anfitrión: <http://mynifi>
- Puerto: 8080 (¡Este es el puerto dentro de la red, no el que mapeamos externamente!)

### Parte 8: Apache Airflow a MinIO

Dentro del servicio Airflow en su navegador en <http://localhost:8085/admin/> configure una nueva conexión en Admin -> Conexiones GUI:

- ID de conexión: myminio_connection
- Tipo de conexión: S3 Dado que no hay un tipo de conexión NiFi, usamos postgres como sustituto.
Extra: consiste en el JSON a continuación:

```
{
  "aws_access_key_id":"minio_admin",
  "aws_secret_access_key": "minio_contraseña",
  "anfitrión": "http://myminio:9000"
}
```

### Revisar

Aprendimos:

- cómo configurar nuestros servicios,
- cómo conectar nuestros servicios entre sí,
- cómo hacer uso de la infraestructura general y
- cómo los servicios pueden interactuar y comunicarse entre sí.

## Construyamos una pipeline ETL con Airflow y NiFi

1. Crearemos una tabla e importaremos algunos datos para procesar en una tabla PostgreSQL
1. Un Airflow DAG programado ejecuta una tarea preparatoria
1. Airflow activa un procesador en Apache NiFi
1. NiFi ejecuta un proceso ETL, que lee la tabla de la base de datos y almacena el resultado en S3 (MinIO)
1. El flujo de aire espera a que NiFi termine,
1. El flujo de aire continúa con alguna otra tarea.

### Datos iniciales de Postgre

1. Abra pgAdmin en: <http://localhost:5050/> con la contraseña maestra que creó anteriormente
1. Navegue por el panel de árbol de la izquierda, abra su servidor postgres_db, abra la entrada de la base de datos y haga clic en la base de datos "postgres".
Con la herramienta Consulta (dentro del menú de herramientas), copie y pegue el contenido de [postgres/create_table.sql](postgres/create_table.sql) para crear la tabla inicial y presione "Ejecutar" (el botón de reproducción)
1. Verifique su tabla recién creada dentro del nodo "Esquemas -> público -> Tablas" en el panel izquierdo, haga clic derecho en la tabla "land_registry_price_paid_uk" ​​y seleccione "Importar/Exportar"
1. Cambie la ventana emergente a "Importar"
1. Para ingresar el nombre del archivo, haga clic en "..." y elija el archivo "pp-monthly.csv"
1. Encabezado, elija "Sí" y presione OK. Esto debería importar 126641 filas a "land_registry_price_paid_uk"

Fuente: <https://www.postgresql.org/message-id/attachment/92675/land-registry-data.txt>

Ahora que tenemos algunos datos transaccionales para ETL, construyamos nuestra canalización

### Pensamientos

Si bien NiFi tiene la opción de programar procesadores con cadenas CRON, generalmente es una mala idea programar trabajos dentro de NiFi, a menos que no tenga otra opción. Con Airflow, podemos monitorear y programar todas nuestras tareas, donde sea que estén, en un solo lugar con una interfaz simple y atractiva, acceso a registros y pipelines de programación altamente personalizables que pueden interactuar, incluir/excluir o depender de El uno al otro.

Del mismo modo, aunque Airflow también puede ejecutar tareas ETL (por ejemplo, codificadas en Python) que idealmente deberían implementarse en NiFi, realmente no debería usar Airflow para hacerlo. Por un lado, Airflow está diseñado para ser un software de monitoreo y programación; por otro lado, perderíamos todas las ventajas inherentes de NiFi con respecto a la extracción, transformación y carga de datos.

Uso de Airflow únicamente como esquema
Más duro y dejando que NiFi haga el trabajo pesado en nuestro backend, obtenemos lo mejor de ambos mundos: la capacidad de Airflow para crear, programar y monitorear flujos de trabajo con los gráficos dirigidos escalables, potentes y altamente configurables de NiFi para enrutar y transformar datos.

### Configuración NiFi

Necesitamos tres procesadores:

- Un procesador QueryDatabaseTable para actuar como un nodo de inicio de nuestro pipeline que se puede activar desde Airflow. Esto devolverá las filas de la tabla PostgreSQL que creamos anteriormente, en formato Avro.
- Un procesador ConvertAvroToParquet para transformar las filas en Parquet
- Un proceso PutS3Object para escribir esos registros en S3 (MinIO)
- Un procesador UpdateAttribute para actuar como el nodo final de nuestro pipeline cuyo estado puede ser consultado por Airflow.

Construyamos nuestra primera tubería NiFi, usando estos procesadores:

![images/nifipipeline.png](images/nifipipeline.png "Pipeline")

#### Crear el nodo de inicio

Diríjase a <http://localhost:8091/nifi/>, luego haga doble clic en el grupo de procesos que creamos originalmente en (#part-4-apache-nifi-to-the-postgres-database), puede crear un nuevo grupo de procesos, pero verifique dos veces que "Servicios de controlador" (dentro de la configuración del grupo de procesos) tenga el servicio "DBCConnectionPool" en la lista y habilitado.
![images/ControllerServices.png](images/ControllerServices.png "Pipeline")

1. Arrastre el primer icono a la placa NiFi, creando así un nuevo Procesador
1. Elija el tipo "QueryDatabaseTable" y haga clic en "agregar"
1. Haga doble clic en el nodo que acaba de crear, cambie a la pestaña "Programación" en la configuración de ese procesador, configure la ejecución en "Nodo principal" y Ejecute la programación en "60 segundos"

![images/scheduling.png](images/scheduling.png "Processor")

1. Cambie a la pestaña "Propiedades" en la configuración de ese procesador y configure estas propiedades:

- Agrupación de conexiones de base de datos: DBCPConnectionPool
- Tipo de base de datos: PostgreSQL
- Nombre de la tabla: land_registry_price_paid_uk

![images/QueryDatabaseTable.png](images/QueryDatabaseTable.png "QueryDatabaseTable")

#### Crear el procesador de conversión

1. Arrastre el primer icono a la placa NiFi, creando así un segundo procesador nuevo
1. Elija el tipo "ConvertAvroToParquet" y haga clic en "agregar"

#### Crear el nodo PutS3Object

1. Cree un procesador PutS3 con esta configuración en la pestaña de propiedades:

- Bucket: miniobucket
- Archivo de credenciales: /opt/nifi/nifi-current/credentials/credentials.properties (la ruta como se especifica dentro del contenedor)
- **URL de anulación de punto final**: <http://myminio:9000> (anulando el punto final de AWS S3 con el de nuestra instancia MinIO privada)

![images/PutS3ObjectConfig.png](images/PutS3ObjectConfig.png "PutS3ObjectConfig")

#### Crear el nodo final

1. Arrastre el primer icono a la placa NiFi, creando así un nuevo Procesador
1. Elija el tipo "UpdateAttribute" y haga clic en "agregar"
1. Haga doble clic en el nodo que acaba de crear y cambie el nombre a "EndNode"
1. Cambie a la pestaña "Propiedades" en esa configuración del procesador, establezca el estado de la tienda en "Estado de la tienda localmente"

Ahora tenemos todo lo que necesitamos en NiFi: un procesador de nodo de inicio, un procesador de nodo final y lo que queramos incluir entre los dos: tareas de extracción, transformación y/o carga de datos.

Continúe, y arrastrando las flechas en cada procesador, enlácelos para que se vean así:
![images/nifipipeline.png](images/nifipipeline.png "Pipeline")

Cuando conecta los enlaces, debe elegir "Para relación: éxito" ("For Relationship: success")

Repare cualquier procesador con un icono de advertencia naranja:
![images/warning.png](images/warning.png "Warning"). La advertencia le dirá lo que falta, por ejemplo, para "La falla de la relación no está conectada a ningún componente" ("Relationship failure is not connected to any component"), simplemente configúrelo para que finalice automáticamente:

![images/relationship.png](images/relationship.png "Relationship")

Una vez que todas las advertencias estén arregladas, elíjalas todas (Ctrl-A o comando-a) y haga clic en el botón "Iniciar" (reproducir) a la izquierda de su pantalla. Esto consultará la base de datos, convertirá el conjunto de resultados avro en parquet y guardará el resultado en MinIo.

![images/minioresult.png](images/minioresult.png "minioresult")

#### Orqueste la pipeline desde Airflow

Haga clic en el procesador QueryDatabaseTable que creamos anteriormente y copie la identificación del procesador.
![images/processorId.png](images/processorId.png "processorId")

Activaremos ese procesador desde Airflow ahora. Abra el archivo "airflow/dags/hello_nifi.py" en el código que clonó de git y, en la línea 17, reemplace la identificación ficticia del procesador con el valor que acaba de copiar en el paso anterior.

1. Abra la consola de Airflow en <http://localhost:8085/admin/> y busque el dag "hello_nifi". Asegúrese de que esté activado,
1. Abra el DAG y vaya a la pestaña "Código", asegúrese de que su identificación haya sido reemplazada.
1. Utilice el botón "activar DAG" para iniciar el ciclo.

Verá que el Airflow DAG activa el flujo nifi:
![images/triggerDAG.png](images/triggerDAG.png "triggerDAG")