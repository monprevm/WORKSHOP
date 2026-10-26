# ASCENSO
### Documento de Diseño de Juego (GDD) — v1.0

| Campo | Valor |
|---|---|
| **Título provisional** | Ascenso |
| **Género** | Plataformas verticales / endless climber / arcade de puntuación |
| **Plataformas** | PC (Web / Steam), Android / iOS |
| **Motor recomendado** | Godot 4.3 (alternativa: Unity 6) |
| **Modo** | Un jugador, partidas de 2 a 5 minutos |
| **Cámara** | 2D ortográfica, scroll vertical automático |
| **Resolución de diseño** | 720 × 1280 (9:16), escalado a 1080 × 1920 |
| **Público** | 12+, casual-core, fans del *score attack* |
| **Precio** | Free-to-play sin paywalls, o premium 2,99 USD |
| **Idiomas** | Español / Inglés |
| **Estado** | Borrador de diseño v1.0 — listo para prototipo |
| **Fecha** | 26/10/2026 |

---

## 1. Resumen ejecutivo

**Ascenso** es un juego de plataformas verticales infinitas en el que el jugador sube por islas flotantes generadas al azar. **No existe botón de salto**: al tocar una plataforma, el personaje rebota automáticamente. El jugador solo decide **izquierda o derecha**, y debe encadenar aterrizajes mientras la cámara sube sola y el mundo se acelera.

> **Frase de venta:** *"Sube o cae. Sin botón de salto, sin freno: solo izquierda y derecha mientras el cielo se acelera bajo tus pies."*

**Gancho:** la tensión nace de una decisión binaria. Cada salto es una apuesta de 0,7 segundos: elegir el lado correcto, corregir en el aire y aterrizar antes de que la plataforma desaparezca por abajo.

**Diferenciador:** la velocidad no solo aumenta la dificultad, también **distorsiona la percepción** (paralaje, balanceo de cámara, espejismos), pero sin romper la regla de oro: *el hitbox nunca miente*.

**Objetivo del jugador:** subir la mayor cantidad de plataformas posible sin morir y superar su récord.

---

## 2. Ficha técnica y alcance

| Aspecto | Definición |
|---|---|
| **Perspectiva** | 2D lateral vertical, cámara fija en X |
| **Loop de sesión** | Partida corta → muerte → récord → reintento inmediato |
| **Duración objetivo de run** | 60–180 s (récord personal: 3–5 min) |
| **Controles** | 2 direcciones, pausa y confirmar |
| **Física** | Timestep fijo a 120 Hz, render a 60 fps |
| **Persistencia** | Récord local, top 5, semilla diaria, ajustes |
| **Contenido MVP** | 1 escenario, 5 tipos de plataforma, 7 fases de velocidad, 3 skins |
| **Accesibilidad** | Remapeo, alto contraste, movimiento reducido, modo calma |
| **Online** | Solo ranking diario (post-MVP), sin multijugador |

**Fuera de alcance (v1.0):** combate, enemigos, coleccionables, tienda, narrativa extensa, cooperativo, editor de niveles.

---

## 3. Pilares de diseño

1. **Control mínimo, decisión máxima.**
   Dos teclas deben bastar para todo. La profundidad viene de la anticipación, no de la cantidad de verbos.

2. **Todo error es legible y justo.**
   Si el jugador muere, debe poder señalar exactamente qué hizo mal. La generación procedural nunca crea una situación imposible.

3. **La velocidad es el antagonista.**
   No hay enemigos: el tiempo y la percepción son el enemigo. Cuanto más alto subes, más rápido y más confuso se vuelve todo.

4. **Reintentar debe doler menos que perder.**
   De la muerte al nuevo salto: menos de 1 segundo, sin menús intermedios.

---

## 4. Referencias

| Referencia | Qué tomamos |
|---|---|
| **Doodle Jump** | Rebote automático, scroll vertical, control horizontal |
| **Icy Tower** | Sensación de ascenso infinito y puntuación por altura |
| **Jump King** | Valor de la precisión y la responsabilidad del jugador |
| **Super Hexagon** | La velocidad como experiencia y la estética de urgencia |
| **Alto's Odyssey** | Paleta por altitud, atmósfera y audio reactivo |

**Lo que NO tomamos:** vidas múltiples, power-ups permanentes, scroll manual, retroceso de cámara.

---

## 5. Público y contexto de juego

- **Jugador primario:** 16–35 años, juega en móvil en sesiones de 2–5 minutos, le gustan los récords y los juegos "un intento más".
- **Jugador secundario:** jugador de teclado en navegador, busca arcade puro y comparación de puntuaciones.
- **Contexto:** partidas rápidas, una mano en móvil, teclado en PC, sin necesidad de audio para jugar.
- **Expectativa emocional:** tensión creciente → alivio tras un aterrizaje ajustado → pánico controlado en las fases altas.

---

## 6. Loop de juego

### 6.1 Loop central (segundos)

```
Observar la próxima plataforma
        ↓
Elegir lado (izquierda / derecha)
        ↓
Corregir en el aire
        ↓
Aterrizar → rebote automático
        ↓
+1 punto / combo / feedback
        ↓
La cámara sube un poco más rápido
        └──────────────► repetir
```

### 6.2 Loop de sesión (minutos)

```
Jugar → morir → ver resultado (2 s) → Reintentar (1 tecla) → nueva run
```

### 6.3 Loop de progresión (días)

- Superar el récord personal.
- Desbloquear skins y estelas por hitos (25 / 50 / 100 / 200 plataformas).
- Reto diario con semilla fija y ranking.
- Completar "medallas" opcionales: 10 aterrizajes perfectos, sobrevivir a la fase 6, etc.

---

## 7. Controles

| Acción | Teclado | Táctil | Mando |
|---|---|---|---|
| Mover a la izquierda | `←` / `A` | Mitad izquierda de la pantalla | D-pad izq. / stick |
| Mover a la derecha | `→` / `D` | Mitad derecha de la pantalla | D-pad der. / stick |
| Pausa | `Esc` / `P` | Botón superior | Start |
| Confirmar / reintentar | `Espacio` / `Enter` | Tocar | A |
| Volver | `Esc` | Botón atrás | B |

**Reglas de input**

- Entrada digital, sin curva de aceleración: al pulsar se fija la velocidad objetivo; la aceleración la aplica la física.
- Zona táctil: 40 % inferior de la pantalla, dividida en dos; no se tapa con el pulgar al personaje.
- Zona muerta de stick: 0,25.
- Latencia objetivo: < 80 ms en móvil, < 40 ms en PC.
- No hay buffer de salto (el salto es automático), pero sí **0,12 s de buffer de dirección** al aterrizar para evitar pulsaciones perdidas.
- Al perder el foco de la ventana, pausa automática.

---

## 8. Reglas del juego

1. El jugador **rebota automáticamente** al tocar la superficie de una plataforma.
2. Solo puede controlar el movimiento **horizontal**.
3. No existe salto manual, doble salto ni dash.
4. Las plataformas son **de una sola dirección**: se atraviesan desde abajo y solo colisionan al caer sobre ellas.
5. La cámara **sube sola y nunca baja**.
6. Caer por debajo de la **línea de muerte** termina la partida.
7. Si el jugador falla un salto pero hay una plataforma más abajo todavía en pantalla, puede **rescatarse** aterrizando en ella. La cámara sigue subiendo, así que el rescate tiene tiempo limitado.
8. Cada plataforma pisada **por primera vez** suma 1 punto.
9. La velocidad de scroll aumenta por fases conforme sube el contador de plataformas.
10. La generación garantiza **al menos una ruta alcanzable** hacia arriba en todo momento.
11. El jugador no puede salir por los lados: los bordes de pantalla actúan como paredes invisibles.
12. La partida termina solo por muerte; no hay límite de tiempo.

---

## 9. Mecánica del jugador

### 9.1 Estados

| Estado | Descripción | Salida |
|---|---|---|
| `IDLE` | En pie sobre plataforma, 0,05 s de anticipación | → `ASCEND` |
| `ASCEND` | Impulso vertical aplicado | → `FALL` |
| `FALL` | Gravedad activa, control aéreo total | → `LAND` o `DEAD` |
| `LAND` | Contacto detectado, squash, +puntos | → `IDLE` |
| `DEAD` | Bajo la línea de muerte | → `GAME_OVER` |

### 9.2 Física

| Parámetro | Valor | Nota |
|---|---|---|
| Gravedad | 2600 px/s² | Constante |
| Impulso de rebote | 1150 px/s | Automático al aterrizar |
| Altura máxima de salto | ≈ 254 px | `v² / 2g` |
| Tiempo de vuelo total | ≈ 0,88 s | Hasta volver a la altura de salida |
| Velocidad horizontal máx. | 460 px/s | Aire y suelo |
| Aceleración horizontal | 3600 px/s² | En suelo |
| Aceleración aérea | 3000 px/s² | Control total en el aire |
| Frenado | 2800 px/s² | Al soltar dirección |
| Retardo de rebote | 0,05 s | Squash + sonido |
| Hitbox | 44 × 52 px | Sprite 56 × 56 |
| Asistencia de borde | 8 px | Imán al aterrizar cerca del borde |
| Tolerancia de penetración | 6 px | Evita falsos negativos |
| Línea de muerte | 80 px bajo el viewport | |
| Gracia bajo la línea | 0,20 s | Solo para perdones visuales |

### 9.3 Sensación

- El salto debe sentirse **elástico y legible**: subida rápida, caída algo más pesada, aterrizaje con squash de 60 ms.
- El personaje se inclina ±12° según la dirección.
- La bufanda/estela indica la velocidad horizontal y ayuda a leer la posición.
- **Nunca** se aplica auto-aim ni corrección horizontal automática: la responsabilidad es del jugador.

### 9.4 Casos límite

| Caso | Comportamiento |
|---|---|
| Aterrizar en el borde exacto | Imán de 8 px + tolerancia de 6 px |
| Atravesar una plataforma desde abajo | Sin colisión; se puede aterrizar al caer |
| Pasar por el lado de una plataforma | Sin colisión lateral |
| Dos plataformas solapadas | Se usa la superficie más alta cruzada en el frame (colisión barrida) |
| Caer muy rápido | Colisión continua por barrido vertical, nunca *tunneling* |
| Cambiar de dirección en el aire | Permitido, con aceleración; no hay giro instantáneo |
| Tocar el borde lateral de la pantalla | Se detiene la velocidad horizontal, sin rebote |

---

## 10. Plataformas

### 10.1 Tipos

| Tipo | Comportamiento | Riesgo | Desde fase |
|---|---|---|---|
| **Común** | Estática, siempre segura | Ninguno | A |
| **Móvil** | Oscila horizontalmente (40–90 px/s, amplitud 60–110 px) | Medio | B |
| **Frágil** | Se rompe 0,30 s después del rebote | Medio | C |
| **Muelle** | Rebote reforzado (1650 px/s, ≈ 523 px de altura) | Bajo (ayuda) | D |
| **Espejismo** | Plataforma fantasma: se desvanece al acercarse, no colisiona | Alto (engaño) | E |

**Reglas de los tipos**

- El **espejismo** nunca puede ser la única plataforma de una banda ni aparecer en los primeros 40 saltos.
- El **muelle** solo aparece si hay una plataforma alcanzable dentro de los 400 px superiores.
- La **frágil** no puede romperse antes de que el jugador la abandone; el rebote siempre se completa.
- La **móvil** nunca se mueve mientras el jugador está en contacto con ella (se detiene al aterrizar) para que el rebote sea predecible.
- Cada tipo tiene **silueta y sonido propios**, no solo color.

### 10.2 Probabilidades por fase

> El porcentaje restante hasta 100 % corresponde a plataformas **comunes**.

| Fase | Móvil | Frágil | Muelle | Espejismo |
|---|---|---|---|---|
| A (0–9) | — | — | — | — |
| B (10–24) | 10 % | — | — | — |
| C (25–49) | 15 % | 10 % | — | — |
| D (50–79) | 20 % | 13 % | 5 % | — |
| E (80–119) | 24 % | 16 % | 5 % | 3 % |
| F (120–179) | 26 % | 18 % | 5 % | 6 % |
| G (180+) | 26 % | 18 % | 5 % | 6 % |

### 10.3 Generación procedural

**Reglas duras**

1. Mantener siempre **4–6 plataformas por encima** del borde superior.
2. Cada banda vertical (slot de 40 px) contiene **1 o 2 plataformas** como máximo; desde la fase C pueden ser 2.
3. Al menos **una plataforma segura cada 2 bandas**.
4. Ninguna plataforma se solapa con otra a menos de 60 px horizontales y 40 px verticales.
5. La siguiente plataforma debe ser alcanzable desde **al menos una** de las dos últimas plataformas.
6. La comprobación de alcance se hace **simulando la trayectoria** con un margen del 15 %.
7. Las primeras 5 plataformas son ancha, centradas, sin peligro y con separación reducida (tutorial encubierto).
8. Semilla guardada por run para reproducir bugs y crear el reto diario.

**Presupuesto de dificultad**

Cada banda recibe un presupuesto `D = 1 + floor(fase / 2)`:

| Elemento | Coste |
|---|---|
| Común | 0 |
| Muelle | 0 |
| Móvil | 1 |
| Frágil | 1 |
| Espejismo | 2 |

Si el tipo elegido supera el presupuesto, se sustituye por una común. El presupuesto evita acumulaciones injustas de peligro.

**Pseudocódigo**

```text
function generateBand(worldY, phase, rng):
    budget = 1 + floor(phase / 2)
    count  = (phase >= C) ? rng.pick(1, 2) : 1

    for i in 0..count-1:
        type = weightedPick(phase, rng)
        if cost(type) > budget:
            type = COMMON
        budget -= cost(type)

        x = clamp(lastX + rng.range(-maxDelta, maxDelta), margin, W - margin)
        w = rng.range(widthMin, widthMax)

        if not isReachable(lastPlatforms, x, w, worldY):
            x = nearestReachableX(lastPlatforms, w, worldY)

        spawn(type, x, worldY, w)

    ensureSafePlatform(phase)
    worldY -= rng.range(gapMin, gapMax)
```

**Limpieza:** las plataformas que quedan 200 px por debajo de la línea de muerte se reciclan en un *pool*.

---

## 11. Scroll, cámara y muerte

### 11.1 Velocidad de scroll

| Fase | Plataformas | Scroll objetivo | Separación vertical | Desfase horizontal |
|---|---|---|---|---|
| A | 0–9 | 70 px/s | 110–150 px | 60–140 px |
| B | 10–24 | 80 px/s | 120–160 px | 70–160 px |
| C | 25–49 | 95 px/s | 130–170 px | 80–180 px |
| D | 50–79 | 112 px/s | 140–185 px | 90–200 px |
| E | 80–119 | 132 px/s | 150–190 px | 100–210 px |
| F | 120–179 | 150 px/s | 160–200 px | 110–220 px |
| G | 180+ | 170 px/s (tope) | 160–200 px | 110–220 px |

- La velocidad se interpola en **3 segundos** al entrar en una fase; nunca da un salto brusco.
- Cada cambio de fase se **telegrafía** 0,8 s antes: líneas de velocidad, *riser* de audio y aviso en el HUD.
- **Tope de 170 px/s** calculado sobre la velocidad de ascenso sostenible (≈ 280–300 px/s en fase F): la cámara nunca puede ganar por pura matemática, solo por errores del jugador.

### 11.2 Regla de alcance de cámara (*catch-up*)

Si el jugador sube por encima del 25 % superior de la pantalla, la cámara acelera hasta **1,8×** su velocidad actual (o hasta la velocidad del jugador + 40 px/s) hasta devolverlo al 35 % de altura.

Esto evita que el jugador se salga por arriba y que las plataformas dejen de generarse a tiempo.

### 11.3 Línea de muerte y rescate

```
┌──────────────────────────────────┐
│  PUNTOS 128        RÉCORD 204    │  HUD
│  VELOCIDAD ▮▮▮▯▯   COMBO ×3      │
│                                  │
│           ▄▄▄▄▄▄                 │  plataformas
│     ▄▄▄▄▄▄         ▄▄▄▄▄▄       │
│            ●                     │  jugador
│    ▄▄▄▄▄▄       ▄▄▄▄▄▄          │
│                                  │
│   zona de rescate (último 15 %)  │  viñeta de peligro
│ ───────── línea de muerte ────── │
└──────────────────────────────────┘
```

- Si el jugador cae en un hueco pero hay una plataforma debajo **todavía visible**, puede aterrizar y salvarse. Esa es la "plataforma previa" del diseño.
- La cámara sigue subiendo, por lo que el rescate solo es posible durante unos segundos; a mayor velocidad de scroll, menor margen.
- La zona inferior del 15 % de la pantalla muestra una viñeta roja pulsante.
- Cruzar la línea de muerte inicia `DEAD` con 0,20 s de gracia visual y luego el *game over*.

---

## 12. Dificultad y curva

### 12.1 Curva esperada

```
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

La dificultad no crece de forma lineal: se escalona por fases, con mesetas para que el jugador pueda "respirar" y percibir su mejora. Cada fase añade **una sola variable nueva** (tipo de plataforma o velocidad), nunca dos a la vez.

### 12.2 Objetivos de balance

| Métrica | Objetivo |
|---|---|
| Primera run mediana | 12–20 plataformas |
| 50 % de jugadores llegan a | 25 plataformas |
| 20 % de jugadores llegan a | 60 plataformas |
| 5 % de jugadores llegan a | 120 plataformas |
| Duración mediana de run | 60–120 s |
| Tasa de reintento tras morir | > 80 % |
| Muertes por "no llegar" (horizontal) | ≈ 60 % |
| Muertes por presión de scroll | ≈ 25 % |
| Muertes por engaño/espejismo | ≈ 15 % |

### 12.3 Anti-frustración

- Reintento instantáneo (< 1 s) con `Espacio` o toque.
- La pantalla de muerte indica **la causa** ("Te quedaste corto", "El vacío te alcanzó", "Un espejismo").
- Sin vidas, sin energía, sin esperas.
- La primera muerte de la sesión muestra un consejo contextual breve (una sola vez).
- Modo **Calma** (accesibilidad): tope de scroll 110 px/s, sin espejismos, ranking separado.

---

## 13. Puntuación y progresión

### 13.1 Puntuación

| Acción | Puntos |
|---|---|
| Plataforma nueva pisada | +1 |
| Aterrizaje perfecto (centro 30 % del ancho) | +2 |
| Racha de 5 perfectos | +5 |
| Racha de 10 perfectos | +10 |
| Racha de 20 perfectos | +20 |
| Récord de altura personal | Bonus visual, sin puntos |

- **Combo:** cada aterrizaje perfecto incrementa el contador; un aterrizaje normal lo reinicia. El multiplicador visual es `1 + 0,1 × combo`, con tope ×2,5.
- **Puntuación final** = puntos + bonus de combo.
- **Altura** se muestra en plataformas y en metros equivalentes (`plataformas × 0,4 m`).
- Se guardan: récord de puntos, récord de plataformas, mejor combo y semilla de la mejor run.

### 13.2 Desbloqueos (solo cosméticos)

| Hito | Desbloqueo |
|---|---|
| 25 plataformas en una run | Skin "Escalador" |
| 50 plataformas | Estela "Chispa" |
| 100 plataformas | Skin "Aurora" |
| 200 plataformas | Estela "Cometa" |
| 10 perfectos seguidos | Título "Pies de pluma" |

Nada de desbloqueos afecta a la física ni a la dificultad.

---

## 14. Feedback y *juice*

| Evento | Feedback |
|---|---|
| Rebote | Squash 0,9 / 1,1 durante 60 ms, sonido suave, polvo |
| Aterrizaje perfecto | *Hitstop* de 40 ms, chispa dorada, nota ascendente |
| Aterrizaje normal | Sonido seco, polvo mínimo |
| Frágil rompiéndose | Crujido, partículas, *shake* de 2–4 px |
| Muelle | Estiramiento, "boing", líneas de velocidad |
| Cambio de fase | *Riser*, flash de 0,15 s, líneas de fondo, vibración corta |
| Combo alto | El tono del rebote sube por semitonos (tope +12) |
| Peligro de caída | Viñeta roja, latido de baja frecuencia |
| Muerte | *Slow-mo* 0,25 s, desaturación, zoom 1,05, silencio de música |

**Límites:** el *screen shake* nunca supera 8 px y nunca afecta a la colisión. Los flashes no superan 3 Hz. Todo el feedback se puede reducir o desactivar.

---

## 15. "Confusión controlada": la velocidad como ilusión

La velocidad debe **marear sin mentir**. Estas son las ilusiones permitidas:

| Recurso | Efecto | Regla de justicia |
|---|---|---|
| Paralaje de 3 capas | El fondo se mueve a 0,2×, 0,5× y 0,8× | Solo visual |
| Balanceo de cámara | ±12 px en seno, desde fase F | No mueve hitboxes |
| Deriva visual de plataformas | ±6 px cosméticos, se anula a 80 px del jugador | La colisión no se mueve |
| Espejismos | Plataformas falsas con brillo tenue | Nunca únicos, siempre desde fase E |
| Inversión de color | Pulso de 0,15 s al cambiar de fase | Desactivable |
| Líneas de velocidad | Refuerzan la sensación de prisa | No exageran la velocidad real |
| Música | BPM 90 → 140, capas que se suman | Avisa antes de acelerar |

**Regla de oro:** *toda confusión es cosmética o está telegrafiada; el hitbox nunca miente.*

---

## 16. UI / UX

### 16.1 HUD

- **Arriba izquierda:** puntos actuales (grande) y récord (pequeño).
- **Arriba derecha:** indicador de velocidad (5 barras) y combo.
- **Sin barra de vida:** no hay vida, solo altura.
- El HUD se desvanece al 40 % de opacidad cuando el jugador está en la mitad superior.
- En móvil, el HUD respeta *safe areas* y no invade la zona táctil.

### 16.2 Pantallas

| Pantalla | Contenido |
|---|---|
| **Menú principal** | Título, Jugar, Diario, Ajustes, Récord, créditos |
| **Tutorial** | 5 plataformas guiadas, textos contextuales de 1 línea, se puede saltar |
| **Partida** | HUD mínimo, sin interrupciones |
| **Pausa** | Continuar, reiniciar, ajustes, salir |
| **Muerte** | Puntos, récord, plataformas, altura, causa, "Reintentar" |
| **Ajustes** | Audio, vibración, alto contraste, movimiento reducido, modo calma, idioma |

### 16.3 Tutorial contextual

| Salto | Texto | Enseña |
|---|---|---|
| 1 | "Salta solo. Tú solo eliges el lado." | Rebote automático |
| 2 | "→ para ir a la derecha" | Control derecho |
| 3 | "← para ir a la izquierda" | Control izquierdo |
| 4 | "Si fallas, busca una plataforma abajo" | Rescate |
| 5 | "El cielo se acelera. Tú también." | Velocidad |

Los textos desaparecen solos a los 3 s y no vuelven a aparecer.

### 16.4 Accesibilidad

- Alto contraste: plataformas con borde blanco de 4 px y fondo oscuro.
- Movimiento reducido: sin *shake*, sin balanceo, sin inversión de color.
- Modo calma: scroll máximo 110 px/s, sin espejismos, ranking propio.
- Sin dependencia del color: cada tipo de plataforma tiene forma e icono distintos.
- Sin audio obligatorio: toda la información crítica es visual.
- Pausa automática al perder el foco.
- Texto escalable y contraste mínimo AA.

---

## 17. Dirección de arte

**Estilo:** vector minimalista, siluetas limpias, neón suave, sin texturas ruidosas. La legibilidad manda: el jugador debe distinguir plataforma, hueco y peligro en menos de 200 ms.

**Paleta por altitud**

| Altura | Cielo | Acento |
|---|---|---|
| 0–25 | Atardecer coral / melocotón | Naranja cálido |
| 26–60 | Crepúsculo violeta | Magenta |
| 61–100 | Noche azul profundo | Cian |
| 101–150 | Aurora verde | Verde lima |
| 151+ | Espacio negro | Blanco estelar |

**Personaje:** 56 × 56 px, silueta oscura con bufanda clara; 8 frames de animación de rebote, 4 de caída, 2 de *squash*.

**Plataformas:** cuerpo oscuro con borde superior luminoso de 4 px y sombra suave. Los tipos peligrosos cambian la forma del borde (grietas, oscilación, brillo fantasma).

**Fondo:** degradado + 3 capas de paralaje (nubes, islas lejanas, partículas). Las capas nunca compiten en contraste con las plataformas.

---

## 18. Audio

**Música:** sistema de 4 capas (pad, arpegio, bajo, percusión) que se activan por fase. BPM de 90 a 140. La transición de fase se anuncia con un *riser* de 0,8 s.

**Efectos**

| Evento | Sonido |
|---|---|
| Rebote | "Thock" corto, tono según combo |
| Perfecto | Campana cristalina |
| Frágil | Crujido de madera |
| Muelle | "Boing" elástico |
| Espejismo | Susurro inverso |
| Velocidad | *Riser* + viento |
| Muerte | Caída de tono + silencio |
| UI | Clics secos y suaves |

**Mezcla:** música a −14 LUFS, efectos prioritarios, *ducking* de 1,5 dB durante avisos de velocidad. Sin audio, el juego debe seguir siendo completamente legible.

---

## 19. Tema y narrativa (ligera)

Un escalador asciende por islas flotantes hacia una señal luminosa en el cielo. No hay texto narrativo: el entorno cuenta la historia mediante el cambio de paleta, los restos de civilizaciones en las capas lejanas y la música. La historia es un premio opcional, nunca un obstáculo para el *loop*.

---

## 20. Estados y flujo

```text
BOOT
  └─► MENÚ ──► TUTORIAL ──► PARTIDA
        │                     │
        │                     ├─► PAUSA ──► PARTIDA
        │                     │
        │                     └─► MUERTE ──► PARTIDA (reintento)
        │                              └──► MENÚ
        └─► AJUSTES / DIARIO / CRÉDITOS
```

| Estado | Entradas | Salidas |
|---|---|---|
| `BOOT` | Carga de assets y guardado | `MENU` |
| `MENU` | Jugar, diario, ajustes | `TUTORIAL` / `PLAYING` |
| `TUTORIAL` | 5 saltos guiados | `PLAYING` |
| `PLAYING` | Input de dirección, pausa | `PAUSED` / `DEAD` |
| `PAUSED` | Continuar, reiniciar, salir | `PLAYING` / `MENU` |
| `DEAD` | *Slow-mo*, resultado | `GAME_OVER` |
| `GAME_OVER` | Reintentar, menú | `PLAYING` / `MENU` |

**Transición de muerte:** 0,25 s de *slow-mo* → panel de resultado → `Espacio`/toque reinicia. Tiempo total percibido: < 1,5 s.

---

## 21. Métricas y telemetría

**Eventos**

- `run_start` (modo, semilla)
- `platform_landed` (índice, tipo, precisión, combo)
- `perfect_land`
- `death` (causa, plataformas, fase, tiempo desde el último aterrizaje, x)
- `speed_tier_changed`
- `retry`
- `session_end`

**KPIs**

| KPI | Objetivo |
|---|---|
| Run mediana | 60–120 s |
| Reintentos por sesión | ≥ 4 |
| Mejor marca en la primera sesión | ≥ 20 plataformas |
| Retención D1 | ≥ 25 % |
| *Crash rate* | < 0,5 % |
| Abandono en el tutorial | < 10 % |

**Protocolo de playtest:** 8–12 personas, 3 sesiones de 15 min, pensar en voz alta, registrar cada muerte y su causa. Ajustar una sola variable por iteración.

---

## 22. Producción

### 22.1 Alcance del MVP

- Loop completo: menú → partida → muerte → reintento.
- Generación procedural con validación de alcance.
- 5 tipos de plataforma y 7 fases de velocidad.
- Puntuación, combo, récord local.
- Tutorial contextual de 5 pasos.
- Teclado + táctil.
- Arte provisional y audio placeholder.
- Ajustes de accesibilidad básicos.

### 22.2 Hitos

| Semana | Entregable |
|---|---|
| 1 | Prototipo de sensación: rebote, control, cámara, muerte |
| 2 | Generación procedural + curva de velocidad |
| 3 | Puntuación, HUD, game over, tutorial |
| 4 | Arte, audio y *juice* |
| 5 | *Playtest*, balance y accesibilidad |
| 6 | Pulido, build web y móvil, *store page* |

### 22.3 Equipo mínimo

- 1 diseñador de juego
- 1 programador
- 1 artista / animador (medio tiempo)
- 1 diseñador de sonido (contrato)
- 1 QA / *playtester* rotativo

### 22.4 Herramientas

Godot 4.3, Aseprite o Illustrator, Audacity/Reaper, Git, Trello o Notion, hoja de cálculo para balance.

---

## 23. Riesgos y mitigaciones

| Riesgo | Probabilidad | Impacto | Mitigación |
|---|---|---|---|
| Generación imposible o injusta | Media | Alto | Validación de alcance por simulación + presupuesto de peligro |
| El scroll supera la capacidad de ascenso | Baja | Alto | Tope de 170 px/s y pruebas de ascenso sostenible |
| La confusión se siente tramposa | Media | Alto | Ilusiones solo cosméticas; hitbox honesto |
| Latencia táctil en móvil | Media | Medio | Zonas grandes, 60 fps, pruebas en dispositivos de gama baja |
| Fatiga visual por velocidad | Media | Medio | Modo calma, movimiento reducido, descansos |
| *Scope creep* (enemigos, tienda) | Alta | Medio | Congelar alcance del MVP; ideas al *backlog* |
| Mareo por balanceo de cámara | Baja | Medio | Desactivable; nunca afecta a la colisión |

---

## 24. Anexo A — Constantes de ajuste

```json
{
  "player": {
    "gravity": 2600,
    "jumpImpulse": 1150,
    "maxHorizontalSpeed": 460,
    "groundAcceleration": 3600,
    "airAcceleration": 3000,
    "brake": 2800,
    "bounceDelay": 0.05,
    "ledgeAssist": 8,
    "hitbox": [44, 52]
  },
  "camera": {
    "baseSpeed": 70,
    "maxSpeed": 170,
    "speedLerpSeconds": 3.0,
    "catchUpMultiplier": 1.8,
    "catchUpTopZone": 0.25,
    "deathLineOffset": 80
  },
  "platforms": {
    "widthMin": 130,
    "widthMax": 230,
    "thickness": 28,
    "gapMin": 110,
    "gapMax": 200,
    "maxCenterDelta": 220,
    "despawnMargin": 200,
    "spawnBuffer": 5
  },
  "scoring": {
    "perfectCenterRatio": 0.30,
    "perfectBonus": 2,
    "comboStep": 5
  }
}
```

> Los valores están en píxeles de la resolución de diseño 720 × 1280 y deben probarse en dispositivo real antes de cerrar el balance.

---

## 25. Anexo B — Checklist de QA

- [ ] El jugador nunca atraviesa una plataforma al caer (colisión barrida).
- [ ] Se puede aterrizar en el borde con asistencia de 8 px.
- [ ] Las plataformas atravesadas desde abajo no bloquean.
- [ ] Ninguna run genera un hueco imposible.
- [ ] El espejismo nunca es la única opción.
- [ ] La cámara no deja al jugador fuera de pantalla por arriba.
- [ ] El scroll nunca supera el tope.
- [ ] El reinicio funciona en < 1 s desde la muerte.
- [ ] El juego pausa al perder el foco.
- [ ] Funciona a 30, 60 y 120 fps sin cambiar la física.
- [ ] La puntuación coincide con las plataformas nuevas.
- [ ] El récord se guarda y se carga correctamente.
- [ ] El modo calma no registra espejismos.
- [ ] Los ajustes de accesibilidad persisten.
- [ ] Sin audio, el juego sigue siendo legible.
- [ ] Sin texto, el tutorial se entiende por iconos.

---

## 26. Anexo C — Glosario

| Término | Definición |
|---|---|
| **Banda** | Franja vertical de 40 px usada para distribuir plataformas |
| **Catch-up** | Aceleración temporal de la cámara cuando el jugador sube demasiado |
| **Combo** | Cadena de aterrizajes perfectos |
| **Espejismo** | Plataforma falsa, sin colisión |
| **Línea de muerte** | Borde inferior que mata al jugador |
| **Ledge assist** | Imán que ayuda a aterrizar en el borde |
| **One-way** | Plataforma que solo colisiona desde arriba |
| **Scroll** | Desplazamiento automático de la cámara |
| **Semilla** | Número que determina la generación de una run |
| **Tope de scroll** | Velocidad máxima que puede alcanzar la cámara |

---

## 27. Decisiones abiertas (para la próxima iteración)

1. ¿El rebote debe ser inmediato o tener 0,05 s de anticipación? (Sensación vs. legibilidad).
2. ¿La muerte debe ser instantánea al cruzar la línea o mantener 0,2 s de gracia?
3. ¿Los espejismos aportan o generan frustración en *playtest*?
4. ¿El modo calma debe compartir récord o tener ranking separado?
5. ¿La velocidad debe topar a los 180 saltos o seguir subiendo con otro recurso de dificultad?
6. ¿Conviene una revancha/revivir opcional o rompe la tensión del *score attack*?

---

*Documento vivo. Cualquier cambio en la física, la curva de velocidad o la generación debe reflejarse aquí y en el prototipo antes de la siguiente sesión de playtest.*
