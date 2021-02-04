# Curso de PostgreSQL 
Este documento contiene información con respecto a los comandos principales de PostgreSQL
# Instalación 
https://www.tecmint.com/install-postgresql-and-pgadmin-in-ubuntu/
## Comandos
### De ayuda
* **\\?:** con el cual podemos ver la lista de todos los comandos disponibles en consola, comandos que empiezan con backslash 
* **\\h:** Con este comando beremos la información de todas las consultas SQL disponibles en consola. Sirve  también para buscar ayuda sobre una consulta específica. 

### De navegación y consulta de información 
* **\\c:** Saltar entre bases de datos 
* **\\l:** Listar base de datos disponibles 
* **\\dt:** Listar las tablas de la base de datos 
* **\\d <nombre_tabla>:** Describir una tabla
* **\\dn:** Listar los esquemas de la base de datos actual
* **\\df:** Listar las funciones disponibles de la base de datos actual 
* **\\dv:** Listar las vistas de la base de datos actual
* **\\du:** Listar los usuarios y sis roles de la base de datos actual
### De inspección y ejecución 
* **\\g:** Volver a ejecutar el comando ejecutado justo antes
* **\\s:** Ver el historial de comandos 
* **\\s <nombre_archivo>:** Si se quiere guardar la lista de comandos ejecutados en un archivo de texto plano  
* **\\i <nombre_archivo>:** Ejevutar los comandos desde un archvo 
* **\\e:** Permite abrir un editor de texto plano, escribir comandos y ejecutar en lote.
* **\\ef:** Equivalente al comando anterior pero permite editar también funciones en PostgreSQL
### Para Debug y optimización 
* **\\timing:** Activar/Desactivar el contador de tiempo por cusulta
### Para cerrar la consola 
* **\\q:** Cerrar la consola

## Particionando tablas 
Esta funcion nos permite realizar una partición de una tabla que contiene mucha infomración, por ejemplo, el siguente ejempli muestra una partición de la bitacora_viaje, donde se creará una partición de la tabla por valores del 2010 al 2021.


```sql
CREATE TABLE bitacora_viaj201001 PARTITION OF bitacora_viaje
FOR VALUES FROM ('2010-01-01') TO ('2021-12-31');

INSERT INTO public.bitacora_viaje(
	id_viaje, fecha)
	VALUES (1, '2010-01-10');

SELECT * FROM bitacora_viaje;
```

## Creación de usario
Se realiza la creación de diferentes roles. Rol con capacidadad de login 
```sql
CREATE ROLE usuario_consulta;
# CREATE USER tambien puede ser usado para crear un rol
#\dg nos permite ver los roles con sus atributos

ALTER ROLE usuario_consulta WITH LOGIN; 
ALTER ROLE usuario_consulta WITH SUPERUSER;

# Creación de contraseña 
ALTER ROLE usuario_consulta WITH PASSWORD '12345';

# Borramos el usuario
DROP ROLE usuario_consulta;
```
## LLaves foráneas

Tienes una tabla de origen, destino y unas acciones
* NO ACTION: No hacer nada
* RESTRICT: Decirle a Postgres que no podemos permitir que la tabla cambie algo.
* CASCADE: Si cambio la tabla de origen, la tabla destino tambien cambia.
* SET NULL quiere decir que nuestra columna en esa fila va a dejar de tener por ejemplo el ID que tenia asociado un 77 y va a convertirse en NULL. Esto por que la tabla destino recibe un cambio y le decimos aPostgres que lo ponga en nulo.
* SET DEFAULT: Si hay un cambio en la tabla origen nuestra tabla destino ponga un valor predeterminado. En un ejemplo un id podra quedar NULL.

## Inserción de datos de forma masiva 
mockaroo.com

## Joins
Cruzar tablas: SQL JOIN teoria de conjuntos con SQL
![alt text](https://ingenieriadesoftware.es/wp-content/uploads/2018/07/sqljoin.jpeg)

## Funciones especiales principales
Funciones espe
* ON CONFLICT DO: Nos ayuda a insertar si ya existe un dato y solo modicarlo 
* RETURNING: Nos permite devolver todos los cambios que hemos hecho sobre la base de datos, no deshace los cambios, si hacemos un insert nos devuelve todos los datos que fueron insertados en la DB
* LIKE / ILIKE: Busqueda especial de infomración 
* IS/ IS NOT: 
### Prueba de ON CONFLICT DO   
```sql
    INSERT INTO public.estacion(
	id, nombre, direccion)
	VALUES (1, 'Nombre', 'Dirección')
	ON CONFLICT(ID) DO UPDATE SET nombre = 'Nombre', direccion = 'Dirección';
```
### Prueba con returning
Al final de la inserción nos regresará el último valor insertado e la DB
```sql
INSERT INTO public.estacion(
	nombre, direccion)
	VALUES ('RET', 'Retorno')
RETURNING *;
```
### Prueba con IS e IS NOT
```sql
SELECT * FROM tren WHERE modelo IS NOT NULL;
```
### Prueba LIKE ILIKE 
LIKE busca entre mayusculas y m
SELECT * FROM pasajero WHERE nombre ILIKE '%o';

### Funciones avanzadas 
* COALESCE: Te permite comparar dos valores y te indica que valor no es nullo
* NULLIF: Te permite comprar dos valores y retorna NULL si son iguales
* GREATEST: Te permite comparar un arreglo de valores y regresa el mayor
* LEAST: Te permite comparar un arreglo de valores y regresa el menor
* Bloques anónimos: Te permite ingresas condicionales dentro de una consulta de DB
```sql
# COALECE
SELECT COALESCE(nombre, 'No Aplica') nombre, direccion_residencia FROM pasajero where id = 1;

# NULLIF
SELECT NULLIF (0,0);

# GREATEST
SELECT GREATEST (0,0,2,15,1,5,2);

# LEAST
SELECT LEAST (0,0,2,15,1,5,2);

# Bloques anónimos 
SELECT id, nombre, direccion_residencia, fecha_nacimiento,
CASE
WHEN fecha_nacimiento > '2000-01-01'THEN 
'Niño'
ELSE 
'Adulto'd
END
FROM pasajero;
```
## Vistas 
Nos sirve para repetir la misma consulta varias veces, existen dos tipos distintos: 
* Vista Volátil: Siempre que hace la consulta a la vista, la base de datos hace la ejecución de la consulta en la BD por lo cuál siempre se mantiene la información actualizada
* Vista materializada: Solo se hace la consulta una vez y esa información queda almancenado en memoria, si la vista materializada no se actualiza, esta podría traer datos viejos que no sirvan. Un ejemplo sería para ver los datos del día de ayer 

### Creación de una vista volátil
Para el siguente ejemplo se crea una vista volátil para consultar quienes son mayores y menores de edad
```sql
 SELECT pasajero.direccion_residencia,
    pasajero.id,
    pasajero.nombre,
    pasajero.fecha_nacimiento,
        CASE
            WHEN (pasajero.fecha_nacimiento > '2001-01-01'::date) THEN 'Niño'::text
            ELSE 'Mayor'::text
        END AS tipo
   FROM pasajero
  ORDER BY
        CASE
            WHEN (pasajero.fecha_nacimiento > '2001-01-01'::date) THEN 'Mayor'::text
            ELSE 'Niño'::text
        END;
```
### Creación de vista materializada
Para la creación de una vista materializada es necesario realizar la recarla la vista matarialiada con el siguente comando: 
```sql
REFRESH MATERIALIZED VIEW despues_noche_mview;
```
```sql
 SELECT viaje.id,
    viaje.inicio,
    viaje.fin,
    viaje.id_pasajero,
    viaje.id_trayecto
   FROM viaje
  WHERE (viaje.inicio > '22:00:00'::time without time zone);
```
Una vez recargada la vista, ya se puede crear y consultar, cabe recalcar que esta vista no será actualizada hasta que el comando REFRESH MATERIALIZED VIEW sea ejecutado. 

## PL/PgSQL (Procedural Languange/PostgreSQL Structured Query Language)
Es un lenguaje imperativo provisto por el gestor que permite ejecutar comandos SQL mediante un lenguaje de sentencias imperativas y uso de funciones, danto mucho más control automático que las sentencias SQL básicas.
Ejemplo: 
![alt text](https://static.platzi.com/media/user_upload/Captura4-96ab9d65-0f83-48c7-b035-b515011ac551.jpg)
```sql
CREATE OR REPLACE FUNCTION llamada_no_interesa(integer) RETURNS integer AS '
DECLARE 
  _llamada_id ALIAS FOR $1;
  _contacto_id integer;

BEGIN

  -- buscar el contacto relacionado
  SELECT _contacto_id INTO _contacto_id FROM llamadas WHERE id = _llamada_id;

  -- actualizar atendida en llamadas
  UPDATE llamadas SET atendida = true WHERE id = _llamada_id;

  -- actualizar no_interesa en contactos
  UPDATE contactos SET no_interesado = true WHERE id = _contacto_id;

  RETURN _contacto_id;

END;
'
```
Declaración de variables en PL/PsSQL y creación de función. Se crea variable rec de tipo record
```sql
CREATE FUNCTION importantePL()
RETURNS void
AS $$
DECLARE 
    rec record; 
    contador integer := 0;
BEGIN
    FOR rec IN SELECT * FROM pasajero LOOP
        RAISE NOTICE 'Un pasajero se llama %', rec.nombre;
        contador := contador + 1;
    END LOOP;
RAISE NOTICE 'Conteo es %',contador;
END
$$
LANGUAGE PLPGSQL;
```
Para mandar a llamar la funcion hacemos lo siguente 
```sql
select importantePL();
# Solo regresará valores en mensajes
```

## Triggers
Para la creación de triggers se debe hacer los siguiente
Crear la función que activará el evento. Para ello se debe tomar los siguientes aspectos:

 * En la declaración de la función, en la sección del retorno se debe indicar que es tipo triggers es decir RETURNS TRIGGER.
Luego indicar en que lenguaje está escrito es decir LANGUAE ‘plpgsql’

* La función tipo triggers debe retornar los valores OLD acepta lo viejo o NEW acepta lo nuevo. Sí se retorna VOID en nuestra función de tipo triggers no aceptamos cambios es decir RETURN NEW;

* Tanto NEW como OLD son un objeto de tipo record y contiene dentro de si el registro, es decir se puede acceder a los campos NEW.campo_nombre del registro

### Creación de trigger
```sql
CREATE OR REPLACE FUNCTION public.impl()
    RETURNS pg_trigger
    LANGUAGE 'plpgsql'
    VOLATILE
    PARALLEL UNSAFE
    COST 100
    
AS $BODY$
DECLARE 
    rec record; 
    contador integer := 0;
BEGIN
    FOR rec IN SELECT * FROM pasajero LOOP
        contador := contador + 1;
    END LOOP;
INSERT INTO conteo_pasajero(total,tiempo)
VALUES (contador, now());
END
$BODY$;
```
```sql
CREATE TRIGGER mitrigger
AFTER INSERT
ON pasajero
FOR EACH ROW 
EXECUTE PROCEDURE impl;
```
## Simulando una conexión a Base de Datos remotas
Instalando la extesion de dblink 
```sql
CREATE EXTENSION dblink;
```
Una vez cargada la extensión se realiza la conexión a la base de datos remota
```sql
SELECT * FROM	
dblink ('dbname=remota
		 port=5432
		 host=localhost
		 user=usuario_consulta
		 password=12345',
	    'SELECT id, fecha FROM vip')
		AS datos_remotos(id integer, fecha date);
```
Se realiza una union de la consulta de la db remota con una local 
```sql
SELECT * FROM pasajero 
JOIN 
dblink ('dbname=remota
		 port=5432
		 host=localhost
		 user=usuario_consulta
		 password=12345',
	    'SELECT id, fecha FROM vip')
		AS datos_remotos(id integer, fecha date)
ON (pasajero.id = datos_remotos.id)
;
``` 
<pre >
Blog de ayuda para PostgreSQL 
https://www.todopostgresql.com/comandos-de-transacciones-en-postgresql/
</pre>

## Transacciones

Una trasacción empaqueta varios pasos en una operación, de forma que se complementen todos todos o ninguno. Los estados intermedios entre los pasos no son visibles para otras transacciones ocurridas en el mismo momento. 
En el aso de que ocurra algùn fallo que impida que se complete la transacciñon, niguno de los pasos se ejecutan y no afectan a los objetos de la base de datos.

Se inicia de la siguente forma: 
```sql
BEGIN
<consultas>
COMMIT | ROLLBACK
```
El siguente ejemplo muestra un caso de error en una insercción, donde el id de public.tren ya existe, por lo que no hará ninguno de los inserts. 
```sql
BEGIN;
INSERT INTO public.estacion(
	nombre, direccion)
	VALUES ('Estación transacción', 'Enrique');

INSERT INTO public.tren(
	id,modelo, capacidad)
	VALUES (6,'Modelo transacción', 123);
COMMIT;
```   
## Extenciones
Activación de extensiones para PostgreSQL
<pre>
https://www.postgresql.org/docs/11/contrib.html
</pre>
Ejemplo de extensión levenshtein 
```sql
-- El nombre de la extensión es fuzzystrmatch: El cual muestra cuantas letras son diferentes y devuelve el entero del total de las letras distintas
CREATE EXTENSION fuzzystrmatch;

SELECT levenshtein('oswaldo','osvaldo');

SELECT difference ('beard','bird');
```

## Mantenimiento
* Vacuum: La más importante, con tres opciones, Vacuum, Freeze y Analyze.
* Full: la tabla quedará limpia en su totalidad
* Freeze: durante el proceso la tabla se congela y no permite modificaciones hasta que no termina la limpieza
* Analyze: solo revisa la tabla

* Analyze: No hace cambios en la tabla. Solo hace una revisión y la muestra.

* Reindex: Aplica para tablas con numerosos registros con indices, como por ejemplo las llaves primarias.

* Cluster: Especificamos al motor de base de datos que reorganice la información en el disco.

https://www.postgresql.org/docs/9.0/maintenance.html