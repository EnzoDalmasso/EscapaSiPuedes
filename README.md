# Escapa Si Puedes

Juego en tercera persona hecho en Unreal Engine 5.3 con Blueprints, como trabajo final de la materia de Unreal (enero a abril de 2025). Arrancás encerrado en una celda y tenés que salir de la cárcel sin que te agarren los policías que patrullan las torres y los pasillos.

Antes de tocar el editor dibujé el mapa entero en un croquis, con dónde va cada guardia, cada llave y cada interruptor:

![Croquis del nivel](Croquis%20TPFinalUnreal.png)

El documento de diseño completo está en [Final_GDD.docx](Final_GDD.docx).

El recorrido es así: buscás una ganzúa escondida en tu celda para abrirla, y después agarrás las llaves que se le caen a un policía para entrar a su oficina. Ahí hay un interruptor que abre la puerta del patio. Cruzás el patio sin que te vean hasta la torre de recepción, la abrís con la misma ganzúa y apretás el botón que abre el portón de salida. Si un guardia te ve te persigue, aunque podés defenderte a golpes.

## Cómo abrirlo

Con Unreal Engine 5.3 instalado, abrir `EscapaSiPuedes.uproject`. Arranca en el menú principal (`Levels/MainMenu`).

Controles: WASD para moverse, mouse para la cámara, Espacio para saltar y E para interactuar con puertas, objetos y el compañero de celda.

## Qué tiene

### IA de los guardias

Fue la parte más larga del proyecto. Cada policía usa un AI Controller (`AIC_EnemyIA`) con AI Perception configurado para vista y daño, y un Behavior Tree (`BT_EnemyIA`) con su Blackboard. El estado está en un enum `E_AIState` (Passive, Investigating, Attacking, Frozen) y el árbol elige rama según ese valor:

- **Passive:** si el guardia tiene una ruta asignada (lo chequea el decorator `BTD_HasPatrolRoute`) la recorre punto por punto con `BP_PatrolRoute`; si no, se queda esperando.
- **Investigating:** va hasta el punto donde percibió algo.
- **Attacking:** fija el foco en el jugador, lo persigue y le pega con un Anim Montage. El daño se aplica con `ApplyDamage`.

Cuando pierde de vista al jugador, un timer en el controller revisa si ya pasó el tiempo para olvidarlo y lo devuelve a Passive. La velocidad también cambia con el estado (caminar al patrullar, correr al perseguir). Los guardias tienen barra de vida flotante y animación de muerte; el modelo y las animaciones son de Mixamo.

Las tareas del árbol están en `Input_IA/Taks/` (sí, con el typo).

### Interacción

Hice una interfaz `BPI_Interaction` y una clase base `BP_InteractItem_Base` de la que heredan dos tipos:

- `BP_Pickup_Parent` para lo que se agarra (ganzúa, llave).
- `BP_OpenCloseSimple_Parent` para puertas, que abren y cierran con una Timeline y detectan de qué lado está el jugador.

Los interruptores (`BP_SwitchFalse`) usan la misma base y animan la palanca al activarse.

### Compañero de celda

Es un NPC con el que se puede hablar. Usa su propia interfaz (`BPI_Dialogo`) y un widget (`BW_Dialogo`) que va mostrando las líneas de un array una por una, con una animación de entrada. Mientras dura el diálogo se desactiva el input del jugador.

### Interfaz

Menú principal (nueva partida, opciones, créditos, salir), pantalla de carga, pausa, Game Over con opción de reintentar o volver al menú, pantalla de victoria al llegar a la salida y barras de vida para el jugador y los enemigos.

## Organización

Cada mecánica la probé primero en un mapa aparte antes de pasarla al nivel final. Esos mapas siguen en el proyecto:

```
Content/EscapaSiPuedes/Levels/
├── MainMenu
├── EscapaSiPuedes              nivel final
├── Iteraciones_Puerta_Llave    pruebas de puertas, llaves e interruptores
├── NPC_CompañeroDeSala         pruebas del diálogo
└── NPC_Enemigo                 pruebas de la IA
```

Los Blueprints propios están en `Content/EscapaSiPuedes/`. Lo que hay en `FirstPerson`, `ThirdPerson`, `LevelPrototyping`, `Infinity_Rogues` y `Police_Offices` son templates de Unreal o packs de assets.
