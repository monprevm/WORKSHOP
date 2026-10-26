# ASCENSO
## Documento de Diseño de Juego — v2.0 (núcleo)

| Campo | Valor |
|---|---|
| **Título** | Ascenso |
| **Género** | Plataformas verticales infinitas / auto-scroll / *score attack* |
| **Plataformas** | Web (prioridad), móvil, PC |
| **Motor** | Phaser 3.90 + JavaScript clásico |
| **Resolución de diseño** | 720 × 1280 (9:16) |
| **Cámara** | 2D ortográfica, scroll vertical automático |
| **Controles** | `←` / `→` (o `A` / `D`), táctil |
| **Sesión** | 2–5 min (varias runs de 20–40 s) |
| **Modo** | Un jugador |
| **Estado** | Diseño de núcleo listo para prototipo |
| **Fecha** | 26/10/2026 |

> **Nota de versión.** Este documento es una **reformulación de núcleo** del `GDD_ASCENSO_v1.0.md`. No lo borra ni lo sustituye: la v1.0 sigue siendo la referencia de sistemas extendidos (tipos de plataforma, combos, reto diario, etc.). La v2.0 define con precisión el bucle principal, el MVP y las constantes que deben implementarse primero.
>
> Las constantes de física, cámara y plataformas coinciden con `src/config.js`. Si el documento y el código divergen, se reporta la diferencia antes de cambiarla. La base técnica decidida del proyecto es **Phaser 3.90**, no Godot: la referencia a Godot en v1.0 §2 está obsoleta.

---

## 0. Resumen de una página

> **En una frase:** no hay botón de salto; solo izquierda, derecha y un cielo que se acelera.

**Ascenso** es un juego de plataformas verticales infinitas. El personaje **rebota automáticamente** al aterrizar; el jugador solo decide hacia qué lado moverse. Las plataformas flotantes aparecen al azar, separadas por un hueco vertical y un desfase horizontal. Si el jugador cae en un hueco y no hay una plataforma debajo que lo rescate, muere. Cuanto más sube, más rápido se desplaza el mundo y más difícil resulta leer la siguiente plataforma.

| Elemento | Definición |
|---|---|
| **Core loop** | Leer la próxima plataforma → elegir lado → corregir en el aire → aterrizar → rebotar → +1 |
| **Objetivo** | Subir la mayor cantidad de plataformas posible sin morir |
| **Fracaso** | Caer al vacío o ser alcanzado por la línea de muerte |
| **Progresión** | Récord personal + desbloqueos cosméticos |
| **Regla de oro** | La velocidad confunde la percepción, pero **el hitbox nunca miente** |

---

## 1. Resumen ejecutivo

**Ascenso** es un arcade de ascenso infinito en el que el jugador sube por islas flotantes generadas proceduralmente. **No existe botón de salto**: cada vez que el personaje toca una plataforma, rebota solo. El jugador únicamente controla el movimiento horizontal, y debe encadenar aterrizajes mientras la cámara sube sin descanso.

> **Frase de venta:** *"Sube o cae. Sin botón de salto, sin freno: solo izquierda y derecha mientras el cielo se acelera bajo tus pies."*

**Gancho:** la tensión nace de una decisión binaria. Cada salto es una apuesta de menos de un segundo: elegir el lado correcto, corregir en el aire y aterrizar antes de que la plataforma desaparezca por abajo.

**Diferenciador:** la velocidad no solo aumenta la dificultad; también **distorsiona la percepción** (paralaje, líneas de velocidad, balanceo visual), pero sin romper la regla de oro: *el hitbox nunca miente*.

**Objetivo del jugador:** subir la mayor cantidad de plataformas posible, superar su récord y sobrevivir un salto más.

---

## 2. Pilares de diseño

1. **Dos teclas, toda la decisión.**
   El juego debe entenderse en cinco segundos y dominarse en veinte partidas. La profundidad viene de la anticipación, no de la cantidad de verbos.

2. **El salto es automático; la responsabilidad es humana.**
   El jugador no pulsa saltar: pulsa hacia dónde quiere ir. Si falla, debe poder señalar exactamente qué hizo mal.

3. **La velocidad confunde, el hitbox no miente.**
   Todo efecto de velocidad es cosmético o está telegrafiado. Nunca se invierten controles, se ocultan plataformas ni se altera la colisión.

4. **Morir es legible; reintentar es instantáneo.**
   De la muerte al nuevo salto: menos de un segundo. Sin vidas, sin esperas, sin menús intermedios.

5. **El récord es la historia.**
   No hay narrativa obligatoria. La altura, la paleta y la música cuentan el viaje; el número final cuenta la habilidad.

---

## 3. Referencias

| Referencia | Qué tomamos |
|---|---|
| **Doodle Jump** | Rebote automático, scroll vertical, control horizontal, plataformas de una sola dirección |
| **Icy Tower** | Sensación de ascenso infinito y puntuación por altura |
| **Jump King** | Valor de la precisión y responsabilidad del jugador |
| **Super Hexagon** | La velocidad como experiencia y estética de urgencia |
| **Alto's Odyssey** | Paleta por altitud, atmósfera y audio reactivo |

**Lo que NO tomamos:** vidas múltiples, power-ups permanentes, scroll manual, cámara que retrocede, enemigos, combate, inventario, checkpoints.

---

## 4. Público y contexto

- **Jugador primario:** 12–35 años, juega en móvil en sesiones de 2–5 minutos, disfruta de récords y del "una más".
- **Jugador secundario:** jugador de teclado en navegador, busca arcade puro y comparación de puntuaciones.
- **Contexto:** partidas rápidas, una mano en móvil, teclado en PC, sin necesidad de audio.
- **Expectativa emocional:** tensión creciente → alivio tras un aterrizaje ajustado → pánico controlado en las fases altas.

---

## 5. Loop de juego

### 5.1 Loop central (segundos)

```text
Observar la próxima plataforma
        ↓
Elegir lado (izquierda / derecha)
        ↓
Corregir en el aire
        ↓
Aterrizar → rebote automático
        ↓
+1 plataforma / feedback
        ↓
La cámara sube un poco más rápido
        └──────────────► repetir
```

### 5.2 Loop de sesión (minutos)

```text
Jugar → morir → ver resultado (2 s) → reintentar (1 tecla) → nueva run
```

### 5.3 Loop de progresión (días)

- Superar el récord personal de plataformas.
- Desbloquear skins y estelas por hitos (25 / 50 / 100 / 200 plataformas).
- Completar medallas opcionales: 10 aterrizajes perfectos, sobrevivir a la fase 6, etc.
- Reto diario con semilla fija y ranking (post-MVP).

---

## 6. Reglas del juego

1. El jugador **rebota automáticamente** al tocar la superficie de una plataforma. No existe botón de salto.
2. Solo puede controlar el movimiento **horizontal**. No hay doble salto, *dash* ni ataque.
3. Las plataformas son **de una sola dirección**: se atraviesan desde abajo y solo colisionan al caer sobre ellas.
4. La cámara **sube sola y nunca baja**.
5. Las plataformas aparecen **aleatoriamente** por encima de la pantalla, con una separación vertical y un desfase horizontal.
6. El jugador no puede salir por los lados: los bordes de la pantalla actúan como paredes invisibles.
7. Cada plataforma pisada **por primera vez** suma 1 punto. Volver a pisar la misma plataforma no suma.
8. Si el jugador falla un salto pero hay una plataforma **más abajo todavía visible**, puede aterrizar en ella y **rescatarse**. La cámara sigue subiendo, así que el rescate tiene tiempo limitado.
9. Si no hay ninguna plataforma debajo, o el jugador cruza la **línea de muerte**, la partida termina.
10. La velocidad de scroll aumenta conforme sube el contador de plataformas.
11. La generación garantiza **al menos una ruta alcanzable** hacia arriba en todo momento.
12. La partida termina solo por muerte; no hay límite de tiempo ni vidas.
13. El hitbox es honesto: ningún efecto visual altera la colisión.

---

## 7. Controles

| Acción | Teclado | Táctil | Mando |
|---|---|---|---|
| Mover a la izquierda | `←` / `A` | Mitad izquierda de la zona táctil | D-pad izq. / stick |
| Mover a la derecha | `→` / `D` | Mitad derecha de la zona táctil | D-pad der. / stick |
| Pausa | `Esc` / `P` | Botón superior | Start |
| Confirmar / reintentar | `Espacio` / `Enter` | Tocar | A |
| Volver | `Esc` | Botón atrás | B |

**Reglas de entrada**

- Entrada **digital**: al pulsar se fija la velocidad objetivo; la aceleración la aplica la física.
- Sin curva de aceleración analógica ni zona muerta en teclado.
- **Buffer de dirección de 0,12 s**: si el jugador pulsa justo antes de aterrizar, la dirección se aplica al rebotar.
- Zona táctil: **40 % inferior** de la pantalla, dividida en dos mitades. No se tapa con el pulgar al personaje.
- Zona muerta de stick: 0,25.
- Latencia objetivo: **< 80 ms** en móvil, **< 40 ms** en PC.
- Pausa automática al perder el foco de la ventana.
- El juego debe ser jugable con una sola mano en móvil y con una sola mano en teclado.

---

## 8. Movimiento y salto

### 8.1 Estados del jugador

> **Supuesto de diseño:** "salta automáticamente" significa que el rebote se dispara al tocar una plataforma; no es un salto periódico en el aire ni un temporizador. El jugador nunca pulsa saltar.

| Estado | Descripción | Salida |
|---|---|---|
| `BOUNCE` | Contacto con la plataforma; squash visual y 0,05 s de anticipación | → `ASCEND` |
| `ASCEND` | Impulso vertical aplicado; control horizontal total | → `FALL` |
| `FALL` | Gravedad activa; control horizontal total | → `LAND` o `DEAD` |
| `LAND` | Contacto detectado; +1 si es plataforma nueva; rebote programado | → `BOUNCE` |
| `DEAD` | Bajo la línea de muerte o sin plataforma de rescate | → `GAME_OVER` |

```text
BOUNCE ──► ASCEND ──► FALL ──► LAND ──► BOUNCE
                         │
                         └──────────────► DEAD ──► GAME_OVER
```

### 8.2 Física

| Parámetro | Valor | Nota |
|---|---|---|
| Gravedad | 2600 px/s² | Constante |
| Impulso de rebote | 1150 px/s | Automático al aterrizar |
| Altura máxima de salto | ≈ 254 px | `v² / 2g` |
| Tiempo hasta el ápice | ≈ 0,44 s | `v / g` |
| Tiempo de vuelo total | ≈ 0,88 s | Hasta volver a la altura de salida |
| Velocidad horizontal máx. | 460 px/s | Aire y contacto |
| Aceleración aérea | 3000 px/s² | Control total en el aire |
| Frenado al soltar | 2800 px/s² | Sin entrada |
| Retardo de rebote | 0,05 s | Squash + sonido; el input sigue activo |
| Hitbox | 44 × 52 px | Silueta visual 56 × 56 |
| Asistencia de borde | 8 px | Imán horizontal al aterrizar cerca del borde |
| Tolerancia de penetración | 6 px | Evita falsos negativos |
| Velocidad máxima de caída | 1600 px/s | Terminal |
| Buffer de dirección | 0,12 s | Al aterrizar |
| Línea de muerte | 80 px bajo el viewport | Con 0,20 s de gracia visual |

### 8.3 Arco de salto

El salto es una parábola fija. La altura `h` se mide entre la **superficie superior** de la plataforma de salida y la **superficie superior** de la plataforma de destino.

```text
h(t) = v₀·t − ½·g·t²
ápice = v₀² / (2g) = 254 px
t_ápice = v₀ / g = 0,44 s
```

| h (px) | Tiempo de aterrizaje | Ascenso medio | Alcance horizontal máximo* |
|---:|---:|---:|---:|
| 110 | 0,776 s | 142 px/s | 321 px |
| 140 | 0,739 s | 189 px/s | 305 px |
| 170 | 0,697 s | 244 px/s | 285 px |
| 200 | 0,647 s | 309 px/s | 262 px |

\* Alcance teórico con velocidad horizontal máxima, descontando la distancia de aceleración (≈ 35 px). Es un límite de diseño, no una promesa de control perfecto.

**Lectura de diseño:** cuanto mayor es el hueco, menos tiempo hay para corregir en el aire, pero más rápido asciende el jugador. El generador debe compensar: a mayor hueco, menor desfase horizontal permitido.

### 8.4 Control horizontal

- La velocidad horizontal se conserva al aterrizar, multiplicada por **0,9**. El jugador puede encadenar zigzags, pero debe gestionar su inercia.
- Acelerar de 0 a 460 px/s tarda **0,153 s** y recorre **35 px**.
- Frenar de 460 a 0 tarda **0,164 s** y recorre **38 px**.
- Invertir de +460 a −460 tarda **0,307 s**; el personaje se desplaza unos **35 px** hacia el lado contrario antes de volver a cruzar su posición original.
- Sin entrada, la velocidad decae a 2800 px/s². No hay auto-aim ni corrección automática.

### 8.5 Aterrizaje

1. Se simula el desplazamiento vertical del frame con **colisión barrida** (nunca *tunneling*).
2. Solo se consideran plataformas cruzadas **de arriba hacia abajo** (`vy < 0`).
3. Si el jugador se solapa horizontalmente con la plataforma, con una asistencia de **8 px** en los bordes, se registra el aterrizaje.
4. Se elige la superficie **más alta** cruzada en ese frame.
5. Se ajusta la posición vertical a la superficie, se aplica el rebote y se conserva el 90 % de la velocidad horizontal.
6. Se marca la plataforma como visitada y se suma 1 punto si es la primera vez.
7. El squash y el sonido se reproducen durante 60 ms; el rebote físico ocurre tras 0,05 s.

### 8.6 Casos límite

| Caso | Comportamiento |
|---|---|
| Aterrizar en el borde exacto | Asistencia de 8 px + tolerancia de 6 px |
| Atravesar una plataforma desde abajo | Sin colisión; se puede aterrizar al caer |
| Pasar por el lado de una plataforma | Sin colisión lateral |
| Dos plataformas solapadas | Se usa la superficie más alta cruzada en el frame |
| Caer muy rápido | Colisión barrida; nunca *tunneling* |
| Cambiar de dirección en el aire | Permitido, con aceleración; no hay giro instantáneo |
| Tocar el borde lateral de la pantalla | Se detiene la velocidad horizontal, sin rebote |
| Aterrizar en la misma plataforma | Rebote, pero sin punto nuevo; la cámara sigue subiendo |
| Caer sin plataforma debajo | `DEAD` al cruzar la línea de muerte |

---

## 9. Plataformas y generación

### 9.1 Geometría

| Propiedad | Valor |
|---|---|
| Ancho | 130–230 px según fase |
| Grosor | 28 px |
| Superficie | Borde superior luminoso de 4 px |
| Colisión | Solo desde arriba (*one-way*) |
| Separación vertical `h` | 110–200 px entre superficies superiores |
| Hueco libre | `h − 28` px (82–172 px) |
| Desfase horizontal | 60–220 px entre centros |
| Margen lateral | 26 px |
| Plataformas por encima de pantalla | 5 |
| Reciclado | 200 px por debajo de la línea de muerte |

> **"Pequeña separación":** el hueco libre entre plataformas es de 82–172 px, siempre menor que la altura máxima de salto (254 px). Nunca hay un salto vertical imposible.

### 9.2 Reglas duras de generación

1. Mantener siempre **5 plataformas por encima** del borde superior.
2. Cada plataforma debe ser alcanzable desde el **centro** de la anterior, con un margen del **15 %**.
3. El desfase horizontal mínimo es **60 px**: no hay plataformas perfectamente alineadas. En las fases altas, el ancho reducido hace que ese desfase exija un movimiento real.
4. El desfase horizontal máximo depende de la fase y del hueco (ver §9.4).
5. No se generan dos plataformas a menos de **40 px verticales** y **60 px horizontales**.
6. Las primeras **5 plataformas** son anchas, cerca del centro, con desfases alternos de 60–80 px y separación reducida: tutorial encubierto.
7. La semilla de la run se guarda para reproducir bugs y crear el reto diario.
8. Si no hay candidato válido, se usa una plataforma de respaldo a 60 px de desfase y con el hueco mínimo de la fase.

### 9.3 Algoritmo de generación

```text
function generateNext(lastPlatform, phase, rng):
    h = rng.range(phase.gapMin, phase.gapMax)
    w = rng.range(phase.widthMin, phase.widthMax)

    // Intentar ambos lados; elegir uno alcanzable.
    candidates = []
    for side in [-1, +1]:
        delta = rng.range(phase.deltaMin, phase.deltaMax)
        x = clamp(lastPlatform.x + side * delta, margin + w/2, W - margin - w/2)
        if isReachable(lastPlatform, x, w, h):
            candidates.push(x)

    if candidates is empty:
        x = fallbackReachableX(lastPlatform, w, h)

    return Platform(x, lastPlatform.y + h, w, h)
```

**Comprobación de alcance**

```text
tLand  = (v0 + sqrt(v0² − 2·g·h)) / g
maxDx  = (vxMax · tLand − vxMax² / (2·accelX)) · 0.85
reach  = abs(target.x − last.x) <= maxDx
```

Con `h = 200` y `maxDx ≈ 262 · 0,85 = 223 px`, el desfase máximo de 220 px queda dentro del 85 % del alcance teórico y deja un **15 % de margen de seguridad**. Con `h = 110`, el margen es mucho mayor. Esta es la garantía matemática de que **no hay saltos imposibles**.

### 9.4 Parámetros por fase

| Fase | Plataformas | Scroll | `h` (px) | Ancho (px) | Desfase (px) | Efecto visual |
|---|---|---:|---:|---:|---:|---|
| A | 0–9 | 70 | 110–150 | 160–226 | 60–140 | Ninguno |
| B | 10–24 | 80 | 120–160 | 150–214 | 70–160 | Paralaje |
| C | 25–49 | 95 | 130–170 | 144–206 | 80–180 | Líneas de velocidad |
| D | 50–79 | 112 | 140–185 | 138–198 | 90–200 | Líneas más densas |
| E | 80–119 | 132 | 150–190 | 134–190 | 100–210 | Balanceo visual |
| F | 120–179 | 150 | 160–200 | 130–182 | 110–220 | Pulso de color |
| G | 180+ | 170 | 160–200 | 128–176 | 110–220 | Todos los efectos |

**Regla de transición:** la velocidad se interpola en **3 segundos** al entrar en una fase; nunca da un salto brusco. Cada cambio se **telegrafía** 0,8 s antes con líneas de velocidad, un *riser* de audio y un aviso en el HUD.

### 9.5 Primeros cinco saltos (tutorial encubierto)

| Salto | Texto | Enseña |
|---|---|---|
| 1 | "Salta solo. Tú solo eliges el lado." | Rebote automático |
| 2 | "→ para ir a la derecha" | Control derecho |
| 3 | "← para ir a la izquierda" | Control izquierdo |
| 4 | "Si fallas, busca una plataforma abajo" | Rescate |
| 5 | "El cielo se acelera. Tú también." | Velocidad |

Los textos desaparecen solos a los 3 s y no vuelven a aparecer. El tutorial se puede saltar. Cada texto se acompaña de un icono (flecha, plataforma, línea de muerte) para que siga siendo legible sin leer.

### 9.6 Variantes de plataforma (ampliación, no MVP)

El MVP usa **solo plataformas comunes**. Las variantes de la v1.0 §10.1 quedan como ampliación y no deben activarse hasta validar el núcleo:

| Variante | Comportamiento | Riesgo | Desde |
|---|---|---|---|
| **Móvil** | Oscila horizontalmente | Medio | Post-MVP |
| **Frágil** | Se rompe 0,30 s después del rebote | Medio | Post-MVP |
| **Muelle** | Rebote reforzado | Bajo | Post-MVP |
| **Espejismo** | Plataforma falsa, sin colisión | Alto | Post-MVP |

La razón es de alcance: añadir variantes antes de que el salto, la cámara y el rescate se sientan bien multiplica los problemas de legibilidad y balance. Para el MVP, ignorar los campos `types` y `two` de `A.PHASES`; activarlos solo después de validar el núcleo.

---

## 10. Scroll, cámara y confusión

### 10.1 Curva de velocidad

La velocidad objetivo depende de las plataformas superadas:

```text
scrollTarget = phase.speed
scrollActual = lerp(scrollActual, scrollTarget, 1 − e^(−dt / 1.2))
```

La constante de tiempo es 1,2 s; el asentamiento visible es de unos **3 segundos**. El parámetro `speedLerpSeconds` de `src/config.js` representa ese asentamiento visible. El jugador percibe la aceleración, pero no recibe un latigazo.

### 10.2 Invariante de sostenibilidad

La cámara **nunca debe ganar por pura matemática**. Para cada fase se compara la velocidad de scroll con el ascenso mínimo sostenible, es decir, el que se obtiene aterrizando siempre en la plataforma más baja permitida (`h = gapMin`):

| Fase | Scroll | Ascenso mínimo | Scroll / ascenso |
|---|---:|---:|---:|
| A | 70 | 142 px/s | 49 % |
| B | 80 | 157 px/s | 51 % |
| C | 95 | 173 px/s | 55 % |
| D | 112 | 189 px/s | 59 % |
| E | 132 | 207 px/s | 64 % |
| F | 150 | 225 px/s | 67 % |
| G | 170 | 225 px/s | 76 % |

**Regla:** `scrollSpeed ≤ 0,80 × minAscent(fase)`. Si un ajuste de balance rompe esta regla, el juego se vuelve imposible por matemática, no por habilidad.

La presión real viene de los errores: fallar un aterrizaje, rescatar en una plataforma baja o perder tiempo corrigiendo en el aire. La cámara es el reloj, no el verdugo.

### 10.3 Cámara y *catch-up*

- La cámara avanza a `scrollActual` y **nunca desciende**; el *catch-up* puede sumar una corrección temporal hacia arriba.
- Si el jugador sube por encima del **25 % superior** de la pantalla, la cámara acelera para devolverlo al **35 %** desde arriba. `base` es la velocidad de scroll actual de la fase (`scrollActual`):

```text
screenY = (player.y − camera.bottom) / H       // 0 = abajo, 1 = arriba

if screenY > 0.75:
    target = base * clamp(1 + (screenY − 0.75) * 3.2, 1, 1.8)
else:
    target = base

camera.speed = lerp(camera.speed, target, 1 − e^(−dt / 0.4))
```

- El multiplicador máximo es **1,8×** sobre la velocidad base. El tope de 170 px/s de §10.1 aplica a la velocidad base, no al *catch-up* temporal.
- **Clamp de seguridad:** si el jugador supera el 8 % superior de la pantalla, la cámara se desplaza lo justo para mantenerlo dentro. Es una corrección de cámara, no de física.
- El *catch-up* no cambia las reglas de muerte ni el hitbox; solo desplaza la cámara y, con ella, la línea de muerte.

### 10.4 Línea de muerte y rescate

```text
┌──────────────────────────────────┐
│  PLATAFORMAS 42     RÉCORD 87    │  HUD
│  VELOCIDAD ▮▮▮▯▯                 │
│                                  │
│           ▄▄▄▄▄▄                 │  plataformas
│     ▄▄▄▄▄         ▄▄▄▄▄▄        │
│            ●                     │  jugador
│    ▄▄▄▄▄       ▄▄▄▄▄▄           │
│                                  │
│   zona de rescate (último 15 %)  │  viñeta de peligro
│ ───────── línea de muerte ────── │  80 px bajo el viewport
└──────────────────────────────────┘
```

- Si el jugador cae en un hueco pero hay una plataforma **debajo y todavía visible**, puede aterrizar y salvarse. Esa es la "plataforma previa" del diseño.
- La cámara sigue subiendo, por lo que el rescate solo es posible durante unos segundos; a mayor velocidad de scroll, menor margen.
- La zona inferior del **15 %** de la pantalla muestra una viñeta roja pulsante.
- Cruzar la línea de muerte inicia `DEAD` con **0,20 s de gracia visual** y luego el *game over*.
- Las plataformas que quedan **200 px por debajo** de la línea de muerte se reciclan. Una plataforma fuera de pantalla **no** cuenta como rescate: para el aterrizaje solo cuentan las plataformas visibles.

### 10.5 "Confusión honesta": la velocidad como ilusión

La velocidad debe **marear sin mentir**. Estos son los recursos permitidos:

| Recurso | Efecto | Regla de justicia |
|---|---|---|
| Paralaje de 3 capas | El fondo se mueve a 0,2×, 0,5× y 0,8× | Solo visual |
| Balanceo de cámara | ±12 px en seno, desde fase E | No mueve hitboxes |
| Líneas de velocidad | Refuerzan la sensación de prisa | No exageran la velocidad real |
| Pulso de color | Flash de 0,15 s al cambiar de fase | Desactivable |
| Estelas de plataforma | Rastro detrás de la plataforma | La plataforma real siempre es la más brillante |
| Viñeta de peligro | Se cierra al acercarse a la línea de muerte | No oculta la plataforma |

**Prohibido:** invertir controles, ocultar plataformas, falsificar la posición del jugador, añadir *input lag*, mover el hitbox con el balanceo o crear plataformas falsas en el MVP. La confusión es **cosmética o telegrafiada**.

---

## 11. Muerte, rescate y *game over*

### 11.1 Causas de muerte

| Causa | Condición | Mensaje |
|---|---|---|
| **Caída al vacío** | No hay plataforma debajo y el jugador cruza la línea de muerte | "Caíste al vacío" |
| **El cielo te alcanzó** | El scroll empuja la línea de muerte por encima del jugador | "El cielo te alcanzó" |
| **Rescate fallido** | El jugador aterriza en una plataforma baja y no logra remontar | "Casi lo logras" |

### 11.2 Secuencia de muerte

```text
Contacto con la línea de muerte
        ↓
0,20 s de gracia visual (slow-mo suave, desaturación)
        ↓
Panel de resultado (0,4 s después)
        ↓
Espacio / toque → nueva run (< 1 s)
```

- Sin menús intermedios, sin anuncios, sin esperas.
- La pantalla de muerte indica la **causa** y una frase breve.
- La primera muerte de la sesión muestra un consejo contextual (una sola vez).

### 11.3 Panel de resultado

| Dato | Ejemplo |
|---|---|
| Plataformas | 42 |
| Récord | 87 |
| Altura | 16,8 m (`plataformas × 0,4`) |
| Mejor racha | 18 |
| Causa | "Caíste al vacío" |
| Semilla | `#A7F3C1` (para reportar bugs) |
| Acciones | Reintentar / Menú |

---

## 12. Dificultad y balance

### 12.1 Curva esperada

```text
Dificultad
   ▲
   │                                 ███
   │                          ███████
   │                   ██████
   │            ██████
   │      █████
   │  ████
   └──────────────────────────────────► Plataformas
      0    20    50    80   120   180
```

La dificultad no crece de forma lineal: se escalona por fases, con mesetas para que el jugador pueda "respirar" y percibir su mejora. La velocidad es la variable que siempre sube; el hueco, el ancho y el desfase se ajustan en pasos pequeños y acumulativos. Ninguna fase introduce dos cambios bruscos a la vez.

### 12.2 Objetivos de balance

| Métrica | Objetivo |
|---|---|
| Primera run mediana | 10–20 plataformas |
| 50 % de jugadores llegan a | 25–40 plataformas |
| 20 % de jugadores llegan a | 60 plataformas |
| 5 % de jugadores llegan a | 120 plataformas |
| Duración mediana de run | 20–40 s |
| Run de récord | 60–120 s |
| Sesión completa | 2–5 min |
| Tasa de reintento tras morir | > 80 % |
| Muertes por fallo horizontal | ≈ 65 % |
| Muertes por presión de scroll | ≈ 25 % |
| Muertes por rescate fallido | ≈ 10 % |

### 12.3 Anti-frustración

- Reintento instantáneo (< 1 s) con `Espacio`, `Enter` o toque.
- La pantalla de muerte explica **la causa**.
- Sin vidas, sin energía, sin esperas.
- Las primeras 5 plataformas son seguras y anchas.
- Modo **Calma** (accesibilidad): tope de scroll 110 px/s, sin balanceo ni líneas de velocidad, ranking separado.
- Ninguna muerte debe ser el resultado de una plataforma inalcanzable.

---

## 13. Puntuación y progresión

### 13.1 Puntuación principal (MVP)

| Acción | Puntos |
|---|---|
| Plataforma nueva pisada | +1 |
| Volver a pisar una plataforma | 0 |
| Aterrizaje perfecto (centro 30 %) | Solo feedback, sin puntos en MVP |

- **Marcador principal:** plataformas.
- **Altura:** `plataformas × 0,4 m` (valor informativo).
- **Récord local:** máximo de plataformas en una run, con semilla y fecha.
- **Estadísticas secundarias:** mejor racha, rescates, porcentaje de aterrizajes perfectos.

> El sistema de perfectos y combos de la v1.0 §13.1 queda como **ampliación**. No se activa hasta que el núcleo sea divertido y legible.

### 13.2 Desbloqueos (solo cosméticos)

| Hito | Desbloqueo |
|---|---|
| 25 plataformas en una run | Skin "Escalador" |
| 50 plataformas | Estela "Chispa" |
| 100 plataformas | Skin "Aurora" |
| 200 plataformas | Estela "Cometa" |

Nada de lo desbloqueado afecta a la física, la generación o la dificultad.

### 13.3 Persistencia

- Récord de plataformas.
- Mejor altura y mejor racha.
- Semilla de la mejor run.
- Ajustes de audio, vibración, movimiento reducido, alto contraste y modo calma.
- Número de runs y tutorial visto.

---

## 14. Feedback y *juice*

| Evento | Feedback |
|---|---|
| Rebote | Squash 0,9 / 1,1 durante 60 ms, sonido suave, polvo |
| Aterrizaje perfecto | *Hitstop* de 40 ms, chispa dorada, nota ascendente |
| Aterrizaje normal | Sonido seco, polvo mínimo |
| Rescate | Destello ámbar, "¡RESCATE!", latido de baja frecuencia |
| Cambio de fase | *Riser*, flash de 0,15 s, líneas de fondo, vibración corta |
| Racha alta | El tono del rebote sube por semitonos (tope +12) |
| Peligro de caída | Viñeta roja, latido de baja frecuencia |
| Muerte | *Slow-mo* 0,25 s, desaturación, zoom 1,05, silencio de música |

**Límites:** el *screen shake* nunca supera **8 px** y nunca afecta a la colisión. Los flashes no superan **3 Hz**. Todo el feedback se puede reducir o desactivar.

---

## 15. Dirección de arte y audio

### 15.1 Arte

**Estilo:** vector minimalista, siluetas limpias, neón suave, sin texturas ruidosas. La legibilidad manda: el jugador debe distinguir plataforma, hueco y peligro en menos de 200 ms.

**Paleta por altitud**

| Altura (plataformas) | Cielo | Acento |
|---|---|---|
| 0–25 | Atardecer coral / melocotón | Naranja cálido |
| 26–60 | Crepúsculo violeta | Magenta |
| 61–100 | Noche azul profundo | Cian |
| 101–150 | Aurora verde | Verde lima |
| 151+ | Espacio negro | Blanco estelar |

**Personaje:** silueta de 56 × 56 px, hitbox 44 × 52, bufanda clara. Animaciones procedurales: 8 frames de rebote, 4 de caída, 2 de *squash*. No se requieren sprites externos para el MVP.

**Plataformas:** cuerpo oscuro con borde superior luminoso de 4 px y sombra suave. El borde superior es siempre la línea más brillante de la pantalla: es la superficie de colisión.

**Fondo:** degradado + 3 capas de paralaje (nubes, islas lejanas, partículas). Las capas nunca compiten en contraste con las plataformas.

### 15.2 Audio

**Música:** sistema de 4 capas (pad, arpegio, bajo, percusión) que se activan por fase. BPM de 90 a 140. La transición de fase se anuncia con un *riser* de 0,8 s.

| Evento | Sonido |
|---|---|
| Rebote | "Thock" corto, tono según racha |
| Perfecto | Campana cristalina |
| Rescate | *Whoosh* ascendente + latido |
| Velocidad | *Riser* + viento |
| Muerte | Caída de tono + silencio |
| UI | Clics secos y suaves |

**Mezcla:** música a −14 LUFS, efectos prioritarios, *ducking* de 1,5 dB durante avisos de velocidad. Sin audio, el juego debe seguir siendo completamente legible.

---

## 16. UI / UX y accesibilidad

### 16.1 HUD

```text
┌──────────────────────────────────┐
│ PLATAFORMAS 42      RÉCORD 87    │
│ VELOCIDAD ▮▮▮▯▯                  │
│                                  │
│           ▄▄▄▄▄▄                 │
│     ▄▄▄▄▄         ▄▄▄▄▄▄        │
│            ●                     │
│    ▄▄▄▄▄       ▄▄▄▄▄▄           │
│                                  │
│   zona de rescate (15 %)         │
└──────────────────────────────────┘
```

- **Arriba izquierda:** plataformas actuales (grande) y récord (pequeño).
- **Arriba derecha:** indicador de velocidad (5 barras).
- **Sin barra de vida:** no hay vida, solo altura.
- El HUD se desvanece al 40 % de opacidad cuando el jugador está en la mitad superior.
- En móvil, respeta *safe areas* y no invade la zona táctil.

### 16.2 Pantallas

| Pantalla | Contenido |
|---|---|
| **Menú principal** | Título, Jugar, Récord, Ajustes, créditos |
| **Tutorial** | 5 plataformas guiadas, textos de una línea, se puede saltar |
| **Partida** | HUD mínimo, sin interrupciones |
| **Pausa** | Continuar, reiniciar, ajustes, salir |
| **Muerte** | Plataformas, récord, altura, causa, "Reintentar" |
| **Ajustes** | Audio, vibración, alto contraste, movimiento reducido, modo calma, idioma |

### 16.3 Accesibilidad

- **Alto contraste:** plataformas con borde blanco de 4 px y fondo oscuro.
- **Movimiento reducido:** sin *shake*, sin balanceo, sin inversión de color.
- **Modo Calma:** scroll máximo 110 px/s, sin balanceo ni líneas de velocidad, récord separado.
- **Sin dependencia del color:** cada tipo de plataforma tiene forma e icono distintos.
- **Sin audio obligatorio:** toda la información crítica es visual.
- **Pausa automática** al perder el foco.
- **Texto escalable** y contraste mínimo AA.
- **Zona táctil** de 40 % inferior, sin tapar al personaje.

---

## 17. Especificación técnica

### 17.1 Stack

| Aspecto | Decisión |
|---|---|
| Motor | Phaser 3.90 |
| Lenguaje | JavaScript clásico, IIFE + `window.Ascenso` |
| Build | Sin *bundler*; scripts clásicos cargados por `index.html` |
| Persistencia | `A.Store` sobre `localStorage` |
| RNG | `A.rng(seed)` (Mulberry32) |
| Audio | `A.audio` (WebAudio procedural) |
| Arte | Procedural; sin assets externos en el MVP |

### 17.2 Bucle y resolución

- Resolución de diseño **720 × 1280**; escalado con `Phaser.Scale.FIT` y respeto de *safe areas*.
- Lógica a **paso fijo de 1/120 s** con acumulador; render a 60 fps con interpolación.
- La física no debe depender de los fps. Probar a 30, 60 y 120 fps.
- Pausa automática con `visibilitychange` / `blur`.

### 17.3 Colisión

- AABB barrido vertical contra plataformas *one-way*.
- Solo colisiona si el jugador desciende (`vy < 0`).
- Se elige la superficie más alta cruzada en el frame.
- Asistencia de borde de 8 px y tolerancia de penetración de 6 px.
- Sin colisión lateral; los bordes de pantalla son paredes.

### 17.4 Generación y *pooling*

- 5 plataformas por encima del borde superior.
- Reciclado de plataformas 200 px por debajo de la línea de muerte.
- *Pool* de 40 plataformas y 200 partículas para evitar asignaciones en el *update*.
- Semilla guardada por run; el *game over* la muestra.

### 17.5 Cámara

- `camera.bottom` avanza con `scrollActual`.
- *Catch-up* según §10.3; nunca desciende.
- El balanceo visual es un **offset de render**, no un cambio de cámara lógica.
- La línea de muerte se calcula desde `camera.bottom`, no desde el sprite.

### 17.6 Pseudocódigo del *update*

```text
update(dt):
    input = readDirection()
    player.vx = approach(player.vx, input * MAX_VX, ACCEL * dt)
    if input == 0:
        player.vx = approach(player.vx, 0, BRAKE * dt)

    player.vy -= G * dt
    prevY = player.y
    player.y += player.vy * dt
    player.x += player.vx * dt
    player.x = clamp(player.x, WALL_L, WALL_R)

    if player.vy < 0:
        landOnHighestPlatform(prevY, player.y)

    updateCamera(dt)
    checkDeath()
```

### 17.7 Anexo de constantes

```json
{
  "design": { "width": 720, "height": 1280, "aspect": "9:16" },
  "player": {
    "gravity": 2600,
    "jumpImpulse": 1150,
    "maxHorizontalSpeed": 460,
    "airAcceleration": 3000,
    "brake": 2800,
    "landingMomentum": 0.9,
    "bounceDelay": 0.05,
    "ledgeAssist": 8,
    "penetration": 6,
    "maxFallSpeed": 1600,
    "hitbox": [44, 52],
    "directionBuffer": 0.12
  },
  "camera": {
    "baseSpeed": 70,
    "maxSpeed": 170,
    "speedLerpSeconds": 3.0,
    "catchUpMultiplier": 1.8,
    "catchUpTopZone": 0.25,
    "catchUpReturn": 0.35,
    "deathLineOffset": 80,
    "deathGrace": 0.2
  },
  "platforms": {
    "thickness": 28,
    "widthMin": 130,
    "widthMax": 230,
    "gapMin": 110,
    "gapMax": 200,
    "maxCenterDelta": 220,
    "marginX": 26,
    "spawnBuffer": 5,
    "despawnMargin": 200
  },
  "scoring": {
    "pointsPerPlatform": 1,
    "perfectCenterRatio": 0.30
  }
}
```

> Los valores están en píxeles de la resolución de diseño 720 × 1280. Deben probarse en dispositivo real antes de cerrar el balance.

---

## 18. Métricas y telemetría

**Eventos**

- `run_start` (semilla, modo)
- `platform_landed` (índice, precisión, tiempo desde el último aterrizaje)
- `rescue` (plataforma, altura restante)
- `death` (causa, plataformas, fase, semilla)
- `speed_tier_changed`
- `retry`
- `session_end`

**KPIs**

| KPI | Objetivo |
|---|---|
| Duración mediana de run | 20–40 s |
| Reintentos por sesión | ≥ 4 |
| Mejor marca en la primera sesión | ≥ 20 plataformas |
| Retención D1 | ≥ 25 % |
| *Crash rate* | < 0,5 % |
| Abandono en el tutorial | < 10 % |

**Protocolo de *playtest*:** 8–12 personas, 3 sesiones de 15 min, pensar en voz alta, registrar cada muerte y su causa. Ajustar una sola variable por iteración.

---

## 19. Producción

### 19.1 Alcance del MVP

- Loop completo: menú → partida → muerte → reintento.
- Generación procedural con validación de alcance.
- Scroll por fases con interpolación y *catch-up*.
- Rescate en plataforma previa.
- Puntuación por plataformas y récord local.
- Tutorial contextual de 5 pasos.
- Teclado + táctil.
- Arte y audio procedurales.
- Ajustes básicos de accesibilidad.

### 19.2 Hitos

| Semana | Entregable |
|---|---|
| 1 | Prototipo de sensación: rebote, control, cámara, muerte, rescate |
| 2 | Generación procedural + curva de velocidad + validación de alcance |
| 3 | Puntuación, HUD, *game over*, tutorial |
| 4 | Arte, audio, *juice* y accesibilidad |
| 5 | *Playtest*, balance, build web y *store page* |

### 19.3 Equipo mínimo

- 1 diseñador de juego.
- 1 programador.
- 1 artista / animador (medio tiempo).
- 1 diseñador de sonido (contrato).
- 1 QA / *playtester* rotativo.

### 19.4 Herramientas

Phaser 3.90, JavaScript clásico, Aseprite o Illustrator, Audacity/Reaper, Git (si se habilita), Notion/Trello y una hoja de cálculo para balance.

---

## 20. Riesgos y mitigaciones

| Riesgo | Probabilidad | Impacto | Mitigación |
|---|---|---|---|
| Generación imposible o injusta | Media | Alto | Validación de alcance por simulación + semilla + *playtest* |
| El scroll supera la capacidad de ascenso | Baja | Alto | Invariante `scroll ≤ 0,8 × minAscent` |
| La confusión se siente tramposa | Media | Alto | Ilusiones solo cosméticas; hitbox honesto |
| Latencia táctil en móvil | Media | Medio | Zonas grandes, 60 fps, pruebas en gama baja |
| Fatiga visual por velocidad | Media | Medio | Modo Calma, movimiento reducido, descansos |
| *Scope creep* (enemigos, tienda) | Alta | Medio | Congelar alcance del MVP; ideas al *backlog* |
| Mareo por balanceo de cámara | Baja | Medio | Desactivable; nunca afecta a la colisión |

---

## 21. Anexos

### Anexo A — Checklist de QA

- [ ] El jugador nunca atraviesa una plataforma al caer (colisión barrida).
- [ ] Se puede aterrizar en el borde con asistencia de 8 px.
- [ ] Las plataformas atravesadas desde abajo no bloquean.
- [ ] Ninguna run genera un hueco imposible.
- [ ] El desfase horizontal mínimo es 60 px.
- [ ] La cámara no deja al jugador fuera de pantalla por arriba.
- [ ] El scroll base nunca supera 170 px/s; el *catch-up* solo actúa por encima del 25 % superior.
- [ ] El scroll base nunca supera el 80 % del ascenso mínimo de la fase.
- [ ] El rescate funciona si hay una plataforma visible debajo.
- [ ] Una plataforma fuera de pantalla no rescata.
- [ ] La causa de muerte mostrada coincide con la real.
- [ ] El reinicio funciona en < 1 s desde la muerte.
- [ ] El juego pausa al perder el foco.
- [ ] Funciona a 30, 60 y 120 fps sin cambiar la física.
- [ ] La puntuación coincide con las plataformas nuevas.
- [ ] El récord se guarda y se carga correctamente.
- [ ] El modo Calma no registra balanceo ni líneas de velocidad.
- [ ] Los ajustes de accesibilidad persisten.
- [ ] Sin audio, el juego sigue siendo legible.
- [ ] Sin texto, el tutorial se entiende por iconos.

### Anexo B — Glosario

| Término | Definición |
|---|---|
| **Auto-rebote** | Salto automático al aterrizar; no hay botón de salto |
| **Catch-up** | Aceleración temporal de la cámara cuando el jugador sube demasiado |
| **Hueco** | Separación vertical entre dos plataformas |
| **Línea de muerte** | Borde inferior que termina la partida |
| **Ledge assist** | Imán que ayuda a aterrizar en el borde |
| **One-way** | Plataforma que solo colisiona desde arriba |
| **Rescate** | Aterrizar en una plataforma inferior visible tras fallar un salto |
| **Scroll** | Desplazamiento automático de la cámara |
| **Semilla** | Número que determina la generación de una run |
| **Fase** | Tramo de dificultad definido por plataformas superadas |

### Anexo C — Decisiones abiertas

1. ¿El rebote debe ser inmediato o mantener 0,05 s de anticipación? (Sensación vs. legibilidad).
2. ¿La muerte debe ser instantánea al cruzar la línea o mantener 0,2 s de gracia?
3. ¿La velocidad horizontal debe conservarse al 100 % o al 90 % al aterrizar?
4. ¿El modo Calma debe compartir récord o tener ranking separado?
5. ¿El sistema de perfectos y combos aporta suficiente profundidad o distrae del objetivo principal?
6. ¿El reto diario entra en el MVP o se pospone?
7. ¿El balanceo de cámara aporta o marea en *playtest*?

---

*Documento vivo. Cualquier cambio en la física, la curva de velocidad o la generación debe reflejarse aquí y en el prototipo antes de la siguiente sesión de playtest.*
