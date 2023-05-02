Pre-requisitos
Para esta sesión va a requerir una cuenta de Amazon Web Services - AWS. Para esto hay varias opciones:

Utilizar una cuenta propia ya creada.

Crear una cuenta nueva. En este caso recuerde que requiere ingresar datos de una tarjeta de crédito. Usaremos recursos disponibles en la capa gratuita (https://aws.amazon.com/free/), pero es posible que se generen algunos costos menores.

Usar la cuenta de AWS Academy enviada a su correo por el instructor. En este caso emplee el Learner Lab.

Ingrese a su cuenta y familiarícese con la consola.

Asegúrese de que en la esquina superior derecha aparezca la región N. Virginia.

Nota: defina un nombre para su grupo, defínalo claramente en reporte y use este nombre como parte inicial de todos los recursos que cree.

Nota 2: la entrega de este taller consiste en un reporte y unos archivos de soporte. Cree el archivo de su reporte como un documento de texto en el que pueda fácilmente incorporar capturas de pantalla, textos y similares. Puede ser un archivo de word, libre office, markdown, entre otros.

Descargue los datos que utilizaremos como fuente
Para este taller usaremos datos de Covid publicados por la OMS.

Visite la página https://covid19.who.int/info/. Navegue a Data y luego a Data Download.

Allí encontrará los datos Daily cases and deaths by date reported to WHO, los cuales puede descargar en formato CSV. El enlace de descarga es https://covid19.who.int/WHO-COVID-19-global-data.csv.

En la página encontrará una descripción de los campos de la tabla. Incluya esta descripción en su reporte.

Note que los códigos de los países están en formato ISO Alpha-2.

Para mapear estos códigos de país a otros códigos, y obtener otra información de cada país usaremos la tabla en formato CSV en el siguiente enlace https://gist.github.com/tadast/8827699#file-countries_codes_and_coordinates-csv.

Para descargar la tabla en formato CSV, de click en Raw y luego click derecho y seleccione Guardar como. El formato por defecto debe ser CSV.

Abra estos dos archivos en Excel o en un editor de texto. Tome un pantallazo e inclúyalo en su reporte.

Configure buckets en S3
En la consola de Amazon AWS, vaya al servicio S3.

Cree un bucket que servirá para los datos de entrada de su Data lake. Como referencia aquí lo llamaremos bucket de entrada.

En este bucket cree una carpeta en la raiz del bucket para los datos de covid, puede llamarse covid. Suba allí el archivo WHO-COVID-19-global-table-data.csv.

Cree otra carpeta en la raíz del bucket para los datos de codigos, puede llamarse paises. Suba allí el archivo countries_codes_and_coordinates.csv.

Cree otro bucket para almacenar los resultados de las consultas. Como referencia aquí lo llamaremos bucket de salida.

Tome un pantallazo de sus buckets e inclúyalo en su reporte.

Conecte los datos con Athena usando Glue
En la consola de Amazon AWS, vaya al servicio Athena.

Athena es un servicio que permite consultar datos almacenamos en S3 usando el lenguaje SQL.

En el panel izquierdo seleccione Workgroups.

Click en Create workgroup. Como nombre del Workgroup use su (primer) nombre (sin repetir existentes).

Como motor de analítica (analytics engine) seleccione Athena SQL. Note que la alternativa es Apache Spark.

En la sección Query result configuration, diligencie el primer campo Location of the query result usando el botón Browse S3 y seleccione el bucket de salida que creó.

Deje los demás campos en sus valores por defecto. Click en Create workgroup en la parte inferior.

De regreso en la consola de Athena, en el panel izquierdo seleccione el ítem Data Sources.

Click en Create Data Source para crear una nueva fuente de datos para consultar.

En su reporte incluya un pantallazo con algunas de las opciones de fuentes de datos que puede incluir.

Seleccione S3 - AWS Glue Data catalog como fuente de datos. Click en Next.

Note que no solamente se selecciona S3 como fuente de los datos crudos, sino que también se selecciona Glue como el servicio para crear un catálogo de datos sobre los datos crudos.

Glue apoya los procesos de ETL y ELT (extracción, transformación, carga) al construir y mantener un catálogo de datos inicialmente crudos (en formato CSV). Este catálogo puede ser luego usado por Athena para realizar las consultas usando SQL sobre los datos en formato CSV.

Seleccione AWS Glue Data catalog in this account ya que el catálogo se construirá en la misma cuenta.

En el método para crear tablas seleccione Create a crawler in AWS Glue.

El Crawler se encargará de recorrer los datos e identificar los campos, tipos, tamaño, etc, para construir el catálogo de datos. Note que también se puede crear una tabla manualmente. En este caso lo haremos con el Crawler.

Click en Create in AWS Glue.

Esto lo llevará a la consola de AWS Glue en una nueva pestaña del navegador.

En el panel izquierdo seleccione Crawlers (Rastreadores).

Click en Create crawler.juan

Ingrese un nombre para el crawler, use su primer nombre-covid. Click en Siguiente.

Selecciones Not yet para indicar que los datos no han sido mapeados a tablas de Glue.

Click en Add a data source. Como fuente seleccione S3, con la opción in this account pues es un bucket S3 en la misma cuenta.

Use el botón Browse S3 para seleccionar el bucket de entrada y la carpeta covid. Click en Choose.

Mantenga las demás opciones en sus valores por defecto y click en Add an S3 data source.

Click en Next.

En Choose an IAM role/Elija un rol de IAM, seleccione el LabRole.

Click en Next/Siguiente.

En Target database seleccione Add database/Añadir una base de datos para crear una nueva base de datos (se abre una nueva pestaña). Defina un nombre, usando su primer nombre-covid-bd, sin tildes, deje los demás campos vacíos y click en Create database/Crear.

Regrese a la configuración del Crawler, refreque la lista de bases de datos y sleccione la que acaba de crear.

En Crawler schedule - Frequency seleccione On demand. En su reporte incluya algunas de las demás opciones (en un pantallazo). Click en Next/Siguiente.

Revise la configuración del crawler, tome un pantallazo, inclúyalo en su reporte, y click en Create Crawler.

Repita los pasos anteriores para agregar un nuevo crawler para el archivo de países. Como nombre utilice sunombre-paises, sin tilde. Para el output, seleccione la misma base de datos del primer crawler, i.e., sunombre-covid-db.

Una vez tenga los dos crawlers creados, vuelva a la consola de AWS Glue, seleccione cada uno y click en Run Crawler para ejecutarlos. Esta operación tardará un poco.

Una vez los crawlers hayan terminado de correr podrá ver las tablas obtenidas en el ítem Tables del panel izquierdo.

Tome un pantallazo de los esquemas de las tablas creadas e inclúyalo en su reporte.

Note que las tablas creadas tienen los nombres de las carpetas que creó en S3, mientras la base de datos tiene el nombre especificado en el Crawler.

Consulte los datos usando SQL desde Athena
Regrese a la consola de Athena.

En el Editor query seleccione su Workgroup (con el nombre definido arriba), como Data source un AwsDataCatalog y como Database la base de datos creada sunombre-covid-db.

En el panel de edición de consultas escriba consultas (una por una) como

    SELECT * FROM "covid" limit 10;
    SELECT * FROM "paises" limit 10;
    SELECT * FROM "covid" where country='Colombia';
    SELECT * FROM "paises" where country='"Colombia"';

y ejecútelas con Run.

Revise los resultados y guarde un pantallazo de cada en su reporte.

En la tabla paises los nombres de los campos son un poco largos. Para modificarlos, vaya a AWS Glue, seleccione la tabla (click en el nombre de la tabla) y en Actions click en Edit schema. Allí sobre cada nombre de campo de doble click y haga los cambios apropiados. Por ejemplo quite “code” de “alpha-2 code” o “(average)” de “latitude (average)”. Click en Save as new table version.

Note que en la tabla paises, los campos vienen con comillas dobles. Para quitarlos e interpretarlos como parte del formato, vaya a AWS Glue, seleccione la tabla y en Actions click en Edit table.

Cambie el Serde serialization lib por org.apache.hadoop.hive.serde2.OpenCSVSerde. En los campos Serde parameters incluya la llave escapeChar con valor ∖, y la llave quoteChar con valor “.

Click en Apply/Save.

En Athena actualice los datos y vuelva a ejecutar la consulta

    SELECT * FROM "paises" limit 10;

¿Qué cambios observa? Incluya la respuesta en su documento de entrega.

Note que ahora puede ejecutar la consulta con una ligera modificación

    SELECT * FROM "paises" where country='Colombia';

Realice un join entre las dos tablas con el siguiente comando

    SELECT * FROM "covid" AS cvd JOIN "paises" AS ps

ON cvd.country=ps.country
LIMIT 5;
¿Qué hace esta consulta? Incluya la respuesta en su reporte. Incluya un snapshot de soporte
