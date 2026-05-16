# Bases de Datos

# Cuarta entrega, Consultas y Procedimientos PL/SQL
#### Hecho por Jaime Castro y Jose Miguel Cenoz 1ºDAW
<img src="portadaE4.png">

#
#
#
#
#
#
#

## Cambios Realizados sobre la tabla

### Modificación de la tabla `JUGADOR`

Se ha añadido un nuevo campo llamado `fecha_guardado` de tipo `DATE`.

#### Cambio realizado

```sql
fecha_guardado DATE
```

### Modelo Relacional y Lógico

Ya que en la entrega anterior no icluímos el modelo lógico dejamos tanto el modelo relacional como el lógico para tener la información necesaria sobre las tablas.

#### Modelo Relacional
<img src="nuevoRelacionalAnual.png">

#### Modelo Lógico
<img src="logicoAnual.png">

## Consultas realizadas sobre las tablas

### Consulta 1
Muestra los jugadores junto con su clase y estadísticas, filtrando únicamente aquellos cuyo nivel sea superior a la media de todos los jugadores. Los resultados se ordenan de mayor a menor nivel.

### Consulta 2
Obtiene el número total de ítems que posee cada jugador, incluyendo aquellos jugadores que no tengan ningún ítem. Además, se deberían mostrar únicamente los jugadores que hayan guardado la partida desde marzo de 2026 en adelante, ordenando el resultado de mayor a menor número de ítems.

### Consulta 3
Calcula la media de probabilidad de drop de objetos para cada enemigo, mostrando todos los enemigos aunque no tengan drops asociados. Los resultados deben ordenarse de mayor a menor probabilidad media.

### Consulta 4
Lista las armas cuyo daño es superior a la media global de todas las armas. Además, se debería mostrar el número de copias disponibles de cada arma (si no tiene copias, mostraría 0), ordenando por daño de forma descendente.

### Consulta 5
Muestra los jugadores junto con su clase, filtrando aquellos cuya fecha de guardado esté dentro de los últimos 30 días respecto a la fecha más reciente registrada en la base de datos. Los resultados deberían aparecer ordenados por fecha de guardado de forma descendente.

```sql
-- =========================================================
-- CONSULTA 1: Jugadores con su clase, estadísticas y filtrado por nivel superior a la media
-- =========================================================

SELECT 
    j.codigoJugador,
    j.nombre,
    j.nivel,
    c.nomClase,
    ej.fuerza,
    ej.destreza,
    ej.arcano,
    ej.salud
FROM JUGADOR j
INNER JOIN CLASE c ON j.codigoClase = c.codigoClase
INNER JOIN EST_JUGADOR ej ON j.codigoJugador = ej.codigoJugador
WHERE j.nivel > (
    SELECT AVG(nivel) FROM JUGADOR
)
ORDER BY j.nivel DESC;


-- =========================================================
-- CONSULTA 2: Número de items por jugador (incluyendo jugadores sin items)
-- =========================================================

SELECT 
    j.codigoJugador,
    j.nombre,
    j.fecha_guardado,
    COUNT(i.codItem) AS total_items
FROM JUGADOR j
LEFT JOIN INVENTARIO inv ON j.codigoJugador = inv.codigoJugador
LEFT JOIN ITEM i ON inv.codigoInventario = i.codigoInventario
WHERE j.fecha_guardado >= TO_DATE('2026-03-01','YYYY-MM-DD')
GROUP BY j.codigoJugador, j.nombre, j.fecha_guardado
ORDER BY total_items DESC;


-- =========================================================
-- CONSULTA 3: Enemigos con media de probabilidad de drop de items
-- =========================================================

SELECT 
    e.codigoEnemigo,
    e.nombre,
    AVG(d.probabilidad) AS prob_media_drop
FROM ENEMIGO e
LEFT JOIN DROP_ENEMIGO_ITEM d 
    ON e.codigoEnemigo = d.codigoEnemigo
GROUP BY e.codigoEnemigo, e.nombre
ORDER BY prob_media_drop DESC NULLS LAST;


-- =========================================================
-- CONSULTA 4: Armas con daño superior al promedio general de armas
-- =========================================================

SELECT 
    a.codItem,
    i.nomItem,
    a.dano,
    COALESCE(ca.numCopias, 0) AS copias
FROM ARMA a
INNER JOIN ITEM i ON a.codItem = i.codItem
LEFT JOIN COPIAS_ARMAS ca ON a.codItem = ca.codItem
WHERE a.dano > (
    SELECT AVG(dano) FROM ARMA
)
ORDER BY a.dano DESC;


-- =========================================================
-- CONSULTA 5: Jugadores con su clase y filtrado por los más recientes
-- =========================================================

SELECT 
    j.codigoJugador,
    j.nombre,
    j.nivel,
    c.nomClase,
    j.fecha_guardado
FROM JUGADOR j
INNER JOIN CLASE c ON j.codigoClase = c.codigoClase
WHERE j.fecha_guardado >= (
    SELECT MAX(fecha_guardado) - 30 FROM JUGADOR
)
ORDER BY j.fecha_guardado DESC;
```

## Programación PLSQL

### Funciones

#### PLSQL_FUNCTION_DANO_TOTAL_JUGADOR

```sql
CREATE OR REPLACE FUNCTION PLSQL_FUNCTION_DANO_TOTAL_JUGADOR (v_codigoJugador jugador.codigojugador%type)
RETURN NUMBER
IS
    v_nombre_jugador jugador.nombre%type;
    v_dano_total arma.dano%type;
BEGIN

    SELECT NVL(SUM(a.dano),0)
    INTO v_dano_total
    FROM JUGADOR j
    INNER JOIN INVENTARIO inv 
        ON j.codigoJugador = inv.codigoJugador
    INNER JOIN ITEM i 
        ON inv.codigoInventario = i.codigoInventario
    INNER JOIN ARMA a 
        ON i.codItem = a.codItem
    WHERE j.codigoJugador = v_codigoJugador;

    DBMS_OUTPUT.PUT_LINE('DAÑO TOTAL DEL JUGADOR: '||v_dano_total);
    RETURN v_dano_total;
    
    EXCEPTION
        WHEN NO_DATA_FOUND THEN
            DBMS_OUTPUT.PUT_LINE('NO SE HA ENCONTRADO EL JUGADOR');
            RETURN 0;
        WHEN OTHERS THEN
            DBMS_OUTPUT.PUT_LINE('SE HAN PRODUCIDO ERRORES');
            RETURN 0;
END;


```

Esta función calcula el daño total de las armas que posee un jugador dentro de su inventario.

La función recibe como parámetro el código del jugador y realiza varias composiciones internas (`INNER JOIN`) entre las tablas `JUGADOR`, `INVENTARIO`, `ITEM` y `ARMA` para obtener todas las armas asociadas al jugador y sumar su daño total mediante la función de agregación `SUM`.

En caso de que el jugador no posea armas, la función devuelve `0` utilizando `NVL`.

La función devuelve un valor numérico de tipo `NUMBER` y un mensaje.



#### PLSQL_FUNCTION_RANGO_JUGADOR

```sql
CREATE OR REPLACE FUNCTION PLSQL_FUNCTION_RANGO_JUGADOR (v_nivel jugador.nivel%type)
RETURN VARCHAR2
IS
    v_rango VARCHAR2(30);
BEGIN

    IF v_nivel < 10 THEN
        v_rango := 'Principiante';

    ELSIF v_nivel BETWEEN 10 AND 20 THEN
        v_rango := 'Intermedio';

    ELSIF v_nivel BETWEEN 21 AND 30 THEN
        v_rango := 'Avanzado';

    ELSE
        v_rango := 'Legendario';
    END IF;

    DBMS_OUTPUT.PUT_LINE('RANGO DEL JUGADOR: ' || v_rango);
    RETURN v_rango;
    
END;


-- Ejemplo de uso:
-- SELECT nombre, nivel, PLSQL_FUNCTION_RANGO_JUGADOR(nivel)
-- FROM JUGADOR;
```


Esta función recibe como parámetro el nivel de un jugador y devuelve un rango descriptivo dependiendo de dicho nivel.

Los rangos definidos son:

- Menor de 10 → `Principiante`
- Entre 10 y 20 → `Intermedio`
- Entre 21 y 30 → `Avanzado`
- Mayor de 30 → `Legendario`

La función devuelve un mensaje con una variable `VARCHAR2` que contiene el valor del rango de los definidos.

### Procedimientos

#### PLSQL_PROCEDURE_GUARDAR_PROGRESO_JUGADOR

```sql
CREATE OR REPLACE PROCEDURE PLSQL_PROCEDURE_GUARDAR_PROGRESO_JUGADOR (v_codigoJugador JUGADOR.codigoJugador%TYPE, v_nuevoNivel JUGADOR.nivel%TYPE)
IS
    v_nivel_actual JUGADOR.nivel%TYPE;
BEGIN

    SELECT nivel
    INTO v_nivel_actual
    FROM JUGADOR
    WHERE codigoJugador = v_codigoJugador;

    IF v_nuevoNivel >= v_nivel_actual THEN

        UPDATE JUGADOR
        SET 
            nivel = v_nuevoNivel,
            fecha_guardado = SYSDATE
        WHERE codigoJugador = v_codigoJugador;

        COMMIT;

        DBMS_OUTPUT.PUT_LINE('PROGRESO GUARDADO CORRECTAMENTE.');

    ELSE
        DBMS_OUTPUT.PUT_LINE('NO SE PUEDE REDUCIR EL NIVEL DEL JUGADOR.');
    END IF;

EXCEPTION

    WHEN NO_DATA_FOUND THEN
        DBMS_OUTPUT.PUT_LINE('JUGADOR NO ENCONTRADO.');

    WHEN VALUE_ERROR THEN
        DBMS_OUTPUT.PUT_LINE('ERROR DE TIPO DE DATO EN LOS VALORES INTRODUCIDOS.');

    WHEN OTHERS THEN
        DBMS_OUTPUT.PUT_LINE('SE HAN PRODUCIDO ERRORES');

END;
```

Este procedimiento permite guardar el progreso de un jugador dentro del sistema.

Recibe como parámetros el código del jugador y el nuevo nivel que se desea asignar.

Su funcionamiento es el siguiente:

1. Se obtiene el nivel actual del jugador desde la base de datos.
2. Se compara el nuevo nivel con el nivel existente.
3. Si el nuevo nivel es mayor o igual, se actualiza el registro del jugador.
4. Se actualiza también la fecha de guardado con la fecha actual del sistema.
5. Se confirma la operación mediante `COMMIT`.
6. Se muestran mensajes informativos mediante `DBMS_OUTPUT`.

En caso de error, el procedimiento controla las siguientes excepciones:
- `NO_DATA_FOUND`: cuando el jugador no existe en la base de datos.
- `VALUE_ERROR`: cuando hay un error de tipo de dato en los valores introducidos.
- `OTHERS`: captura cualquier otro error no controlado, mostrando un mensaje genérico.



#### PLSQL_PROCEDURE_LISTAR_INVENTARIO_JUGADOR

```sql
CREATE OR REPLACE PROCEDURE PLSQL_PROCEDURE_LISTAR_INVENTARIO_JUGADOR (v_codigoJugador JUGADOR.codigoJugador%TYPE)
IS

    CURSOR cur_inv IS
        SELECT 
            i.codItem,
            i.nomItem,
            i.tipoItem,
            inv.tipoInventario
        FROM INVENTARIO inv
        INNER JOIN ITEM i
            ON inv.codigoInventario = i.codigoInventario
        WHERE inv.codigoJugador = v_codigoJugador;

    v_codItem        ITEM.codItem%TYPE;
    v_nomItem        ITEM.nomItem%TYPE;
    v_tipoItem       ITEM.tipoItem%TYPE;
    v_tipoInventario INVENTARIO.tipoInventario%TYPE;

BEGIN

    OPEN cur_inv;

    LOOP
        FETCH cur_inv INTO 
            v_codItem,
            v_nomItem,
            v_tipoItem,
            v_tipoInventario;

        EXIT WHEN cur_inv%NOTFOUND;

        DBMS_OUTPUT.PUT_LINE(
            'ITEM: ' || v_codItem ||
            ' | ' || v_nomItem ||
            ' | TIPO: ' || v_tipoItem ||
            ' | INVENTARIO: ' || v_tipoInventario
        );
    END LOOP;

    CLOSE cur_inv;

EXCEPTION

    WHEN NO_DATA_FOUND THEN
        DBMS_OUTPUT.PUT_LINE('NO EXISTEN DATOS PARA ESE JUGADOR.');

    WHEN VALUE_ERROR THEN
        DBMS_OUTPUT.PUT_LINE('ERROR DE TIPO DE DATO.');

    WHEN OTHERS THEN
        DBMS_OUTPUT.PUT_LINE('SE HAN PRODUCIDO ERRORES');

END;
```

Este procedimiento muestra el inventario completo asociado a un jugador mediante el uso de un cursor explícito.

Recibe como parámetro el código del jugador.

Su funcionamiento es el siguiente:

1. Se define un cursor que recupera los ítems asociados al inventario del jugador.
2. Se abre el cursor para comenzar la lectura de datos.
3. Se recorre cada fila del cursor de forma secuencial.
4. Por cada registro, se muestran los datos del ítem mediante `DBMS_OUTPUT`.
5. Se cierra el cursor una vez finalizada la lectura.

En caso de error, se contemplan las siguientes excepciones:
- `NO_DATA_FOUND`: si no existen datos asociados al jugador.
- `VALUE_ERROR`: si ocurre un problema de tipo de datos.
- `OTHERS`: captura cualquier otro error no previsto, mostrando un mensaje genérico.

### Triggers (Investigación)











## Bases de datos no relacionales. Trabajo de investigación

### ¿Qué es una base de datos no relacional?

Las bases de datos no relacionales, también conocidas como *NoSQL / Not Only SQL*, son sistemas de gestión de bases de datos que no utilizan el modelo tradicional basado en tablas (filas y columnas) ni dependen de relaciones mediante claves foráneas.

En su lugar, emplean modelos de datos más flexibles como:

- Documentos (JSON o BSON)  
- Clave-valor  
- Columnas anchas  
- Grafos  

Su funcionamiento se basa habitualmente en estructuras simples de *clave–valor*, donde la clave identifica el dato y el valor suele ser un JSON. Esto permite una gran flexibilidad y facilita la escalabilidad horizontal.

Sin embargo, esta simplicidad tiene implicaciones: las consultas complejas pueden ser menos eficientes y es necesario diseñar bien cómo se accederá a los datos desde el inicio. Además, al no existir un esquema rígido, una mala organización interna puede generar problemas de consistencia.

Otro aspecto importante es el *particionado interno*, donde cada clave se transforma en un hash que determina en qué servidor se almacena. Esto permite distribuir los datos fácilmente y escalar el sistema añadiendo más nodos.

Cabe destacar que *NoSQL* y *Not Only SQL* no son exactamente lo mismo:
- *NoSQL* → alternativa a las bases de datos relacionales  
- *Not Only SQL* → complemento a las bases de datos relacionales  

Estas bases de datos están diseñadas para manejar grandes volúmenes de información, alta concurrencia y estructuras dinámicas, siendo muy utilizadas en aplicaciones modernas como videojuegos, redes sociales o sistemas distribuidos.

---

### Ventajas y desventajas

| Aspecto                     | Bases de datos relacionales (SQL)                          | Bases de datos no relacionales (NoSQL)                     |
|----------------------------|------------------------------------------------------------|------------------------------------------------------------|
| Modelo de datos            | Tablas con filas y columnas                                | Documentos, clave-valor, grafos o columnas                 |
| Esquema                    | Fijo y predefinido                                         | Flexible y dinámico                                        |
| Relaciones                 | Uso intensivo de claves foráneas (JOINs)                   | Relaciones embebidas o referencias simples                 |
| Escalabilidad              | Vertical (más potencia en un solo servidor)                | Horizontal (distribución en varios servidores)             |
| Rendimiento                | Bueno en consultas complejas                               | Muy alto en grandes volúmenes y consultas simples          |
| Consistencia               | Alta (propiedades ACID)                                    | Variable (modelo BASE en muchos casos)                     |
| Flexibilidad               | Baja                                                       | Alta                                                       |
| Complejidad de consultas   | Baja en consultas complejas (JOINs disponibles)            | Mayor en consultas complejas                               |
| Redundancia de datos       | Baja (normalización)                                       | Mayor (desnormalización)                                   |
| Casos de uso ideales       | Sistemas financieros, ERP, aplicaciones críticas           | Big Data, tiempo real, videojuegos, redes sociales         |

---

### Bases de datos no relacionales más utilizadas

Actualmente, las bases de datos NoSQL más utilizadas son:

- MongoDB  
  - Modelo orientado a documentos  
  - Muy utilizada en aplicaciones web y videojuegos  

- Redis  
  - Base de datos en memoria  
  - Ideal para caché y sistemas en tiempo real  

- Apache Cassandra  
  - Altamente escalable  
  - Usada en sistemas con grandes volúmenes de datos  

- Amazon DynamoDB  
  - Servicio gestionado en la nube  
  - Alta disponibilidad  

- Neo4j  
  - Orientada a grafos  
  - Ideal para relaciones complejas  

Además, cada vez se impulsa más su uso en entornos cloud, con soluciones como DynamoDB, BigTable o CosmosDB.

---

### Elección de base de datos no relacional y justificación

La base de datos no relacional que escogería para migrar el sistema desarrollado sería *MongoDB*.

La elección se justifica por varios motivos adaptados al contexto de un videojuego local (sin funcionalidad online):

En primer lugar, MongoDB permite trabajar con un modelo orientado a documentos (JSON), lo que facilita representar entidades complejas como los jugadores, sus estadísticas y su inventario en una única estructura. Esto simplifica considerablemente el diseño respecto al modelo relacional, donde la información está distribuida en múltiples tablas.

En segundo lugar, al tratarse de un videojuego local, el rendimiento en acceso a datos es clave. MongoDB permite acceder a toda la información de un jugador sin necesidad de realizar múltiples JOINs, reduciendo el tiempo de consulta y mejorando la eficiencia.

Además, la flexibilidad del esquema permite añadir nuevos elementos al juego sin necesidad de modificar la estructura de la base de datos, lo cual es especialmente útil en entornos de desarrollo iterativo como los videojuegos.

Aunque MongoDB está diseñado para sistemas distribuidos, también funciona perfectamente en local, sin necesidad de aprovechar su escalabilidad horizontal.

Por último, su facilidad de uso y la claridad de su estructura en formato JSON lo convierten en una opción adecuada para proyectos de tamaño medio, donde prima la simplicidad y el mantenimiento.

En conclusión, MongoDB es una opción idónea para este caso, ya que permite simplificar el modelo de datos, mejorar el rendimiento y facilitar la evolución del sistema.