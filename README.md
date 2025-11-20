## Getting Started

Welcome to the VS Code Java world. Here is a guideline to help you get started to write Java code in Visual Studio Code.

## Folder Structure

The workspace contains two folders by default, where:

- `src`: the folder to maintain sources
- `lib`: the folder to maintain dependencies

Meanwhile, the compiled output files will be generated in the `bin` folder by default.

> If you want to customize the folder structure, open `.vscode/settings.json` and update the related settings there.

## Dependency Management

The `JAVA PROJECTS` view allows you to manage your dependencies. More details can be found [here](https://github.com/microsoft/vscode-java-dependency#manage-dependencies).

# Bases de Datos 
## Proyecto Anual Segunda Entrega


##### Hecho por Jaime Castro y José Miguel Cenoz, 1ºDAW

<div style="page-break-before: always;"></div>


## Índice

1. [Descripción del Sistema](#descripción-del-sistema)
2. [Requisitos del Sistema](#requisitos-del-sistema)
    - [Funcionales](#funcionales)
    - [No Funcionales](#no-funcionales)
        - [Estudio SGBD a usar](#estudio-SGBD-a-usar)
        - [Funcionamiento de _BBDD_ distribuida](#funcionamiento-de-_BBDD_-distribuida)
        - [Ubicación de los datos o Almacenamiento Físico](#ubicación-de-los-datos-o-almacenamiento-físico)
        - [Crecimiento](#crecimiento)
        - [Plan de respaldo](#plan-de-respaldo)
3. [Bibliografía](#bibliografía)


## Descripción del Sistema
Una compañía de videojuegos ha desarrollado un juego en el que se controla un personaje y tienes que derrotar distintos jefes, ubicados en el mapa del juego. 
Se quiere recoger información del jugador, jefes, ubicaciones del mapa, armas, objetos y enemigos básicos.

## Requisitos del sistema
### Funcionales
El jugador puede elegir distintas clases para su personaje, esta clase determina las estadísticas y armas base del jugador. Más adelante el jugador podrá redistribuir sus estadísticas, pero no cambiar de clase.

Un jugador solo puede tener una clase, pero una clase puede no tener jugador.

El jugador puede guardar varias armas en su inventario. Puede equiparse un arma, o no llevar ninguna.

Las armas son un tipo de objeto, y pueden encontrarse tanto en ciertos puntos del mapa, ser soltadas por enemigos básicos, o por ciertos jefes, que soltarán un arma de jefe.

El jugador puede guardar varias veces la misma arma y mejorarla ya que en el mundo
pueden aparecer de los enemigos una misma copia.

El jugador puede portar hasta tres objetos, también puede no llevar objetos.

El jugador puede guardar varias veces la misma arma y mejorarla ya que en el mundo
pueden aparecer de los enemigos una misma copia.

Existen distintos tipos de objetos; armas arrojadizas, consumibles, los
objetos, pueden ser soltados por los enemigos y jefes (cuando son derrotados), o encontrados en el mapa.

Hay distintos tipos de enemigo repartidos por puntos del mapa, y un único jefe. 
Del jugador interesa conocer; Código de jugador, nombre, clase, estadísticas, armas que porta, objetos que porta, objetos en el inventario.

De los enemigos interesa saber; Código de enemigo, estadísticas, arma que porta, objetos que suelta y probabilidad de que lo suelte.

Del mapa es necesario tener; Código de mapa, puntos en los que hay enemigos y cuales son, ubicación del jefe.


### No funcionales
Se ha seleccionado para esta tarea, una base de datos relacional, ya que los datos de un videojuego, mantienen una relación natural entre entidades participantes del medio.

Se hacen referencias constantes entre tablas, por ejemplo transacciones o reglas (enemigo no deja arma si no es eliminado por jugador, por ejemplo). 

Usando el modelo relacional es posible conseguir diseño, integridad y consultas. Siendo estas últimas un método eficiente para recuperar información sobre la experiencia, por ejemplo si hay zonas demasiado diİciles, comparando las veces que el jugador ha sido eliminado en una zona concreta, cuáles son las armas mas usadas etc.

#### Estudio _SGBD_ a usar
Hay varias opciones a la hora de seleccionar el _SGBD_ que más nos convene para este sistema de información. Entre ellos hemos seleccionado `MySQL`, `SQLite`, `PostgreSQL` y `Oracle`.
- **`MySQL`**: gran facilidad de uso y rednimiento. Tiene soporte multiplataforma, lo que permite trabajar con los archivos del juego fácilmente, y mantiene la comunicación segura entre navegador y servidor mediante su soporte `SSL`. El mayor problema es que no es escalable, por lo que si se quieren realizar cambios, podría no ser la mejor opción.

- **`SQLite`** nos permite guardar toda esta información fácilmente, aunque su tamaño sea mucho menor que cualquier otro al tratarse de una biblioteca, su gran portabilidad y rendimiento nos permite usar este sistema para trabajar con la _BBDD_.

- **`PostgreSQL`**: gran capacidad de uso multiplataforma y herramientas que permiten administrar fácilmente la _BBDD_. Este _SGBD_ es robusto y eficaz, pero al mismo tiempo, nos permite realizar una configuración compatible con _BBDD_ más pequeñas. También, al ser un sistema que permite el uso multiplataforma, podemos manejar `JSON` o `CTE`,(prominentes en el desarrollo de videojuegos), cosa especialmente útil, cuando se trata de sistemas de inventario.

- **`Oracle`**, por otro lado, es más completo. Por lo que sería perfecto si se tratara de una base de datos que estuviera en constante evolución, ya que tiene una gran escalabilidad. Sin embargo, éste no es el caso, por lo que no es el _SGBD_ que más nos conviene, aunque tenga buenas características para uso empresarial.

En conclusión, por todo lo anteriormente descrito, nos decantamos por usar el _SGBD_ de **`PostgreSQL`**; por su capacidad de personalización, manejo de `JSON/CTE`, gran uso multiplataforma etc.
#### Funcionamiento de BBDD distribuida
Esta _BBDD_  funcionaría en una distribuida, aunque no tendría sentido más allá de la seguridad de la información, ya que la _BBDD_ se ejecutaría en la máquina personal del jugador. Sin embargo, si se tratará de un juego en línea, dónde las actualizaciones son constantes, podría ser preferible, para que la máquina del jugador no tenga que ejecutar dicha _BBDD_, puesto que pueden llegar a alcanzar tamaños enormes.

#### Ubicación de los datos o Almacenamiento Físico
La empresa tendría un servidor central para ubicar la base de datos principal del juego, en el que se guardarían absolutamente todos los datos de la misma. Cada jugador tendría su propia copia del mismo, pero con los datos actualizados de su partida. 
Si se hiciera alguna actualización del juego, sería mediante un _script_ de migración, que en resumen es un _script_ _SQL_ que sirve para hacer cambios en una base de datos de forma segura (en el transcurso de actualizaciones). 
Esto se haría en caso de que se añada o elimine algún arma del juego, o para contenido extra del juego.
Cabe mencionar que se puede dar el caso de que haya un riesgo de incompatibilidad de versión, si un jugador no actualiza el juego cuando se distribuya una actualización, su _BBDD_ quedaría desactualizada, por lo que los datos podrían no coincidir con la versión que se publica, o parche actual de la empresa.
#### Crecimiento
No se prevé un gran crecimiento de esta base de datos, sin embargo, si la compañía desea hacer actualizaciones o ampliaciones de contenido (como contenido descargable, o una continuación del mismo). Se podría usar esta _BBDD_ de plantilla.

#### Plan de respaldo
Al tratarse de una base de datos pequeña y que se actualiza poco, no es necesario un plan de respaldo especialmente complejo (solamente que el jugador guarde su partida antes de salir del juego). Sin embargo, también se podría plantear, que la empresa distribuidora del juego permitiera autoguardados en la nube de las partidas de los distintos jugadores, por lo que si un jugador pierde su partida, o esta se corrompe pueda recuperarla fácilmente.

## Modelo Entidad Relación

```mermaid
erDiagram
JUGADOR {
    int id_jugador
    string nombre
    string clase
    int nivel
    }

    JEFE {
        int id_jefe
        string nombre
        int dificultad
    }

    UBICACION {
        int id_ubicacion
        string nombre
        string tipo
    }

    ARMA {
        int id_arma
        string nombre
        int danio
    }

    OBJETO {
        int id_objeto
        string nombre
        string descripcion
    }

    ENEMIGO {
        int id_enemigo
        string nombre
        int nivel
    }

    %% Relaciones
    JUGADOR ||--o{ ARMA : posee
    JUGADOR ||--o{ OBJETO : recoge
    JUGADOR ||--o{ ENEMIGO : combate
    JUGADOR ||--o{ JEFE : derrota
    JEFE }o--o{ UBICACION : aparece_en
    ENEMIGO }o--o{ UBICACION : habita
```


## Bibliografía
Parte de la información que hemos expuesto está en internet y otra en los apuntes de
clase, las fuentes son las siguientes:
- https://www.red-gate.com/blog/a-database-model-for-action-games?.com
- https://learn.microsoŌ.com/en-us/sql/relational-databases/databases/estimate-the-size-of-a-database?view=sql-server-ver17
- http://Postgresql.org
-  https://docs.aws.amazon.com/wellarchitected/latest/games-industry-
- lens/games-scenario-4.html?.com
- https://knowledgebase.apexsql.com/migration-scripts-introduction-and-general-review/?_x_tr_sl&_x_tr_tl&_x_tr_hl
También dejamos acceso a información sobre el juego en el que nos hemos basado para contrastar información sobre el trabajo que estamos realizando.
- https://darksouls.fandom.com/es/wiki/Dark_Souls_III
