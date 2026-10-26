# AGENTS.md — Instrucciones para agentes de IA

> Proyecto: **ASCENSO**, juego web de plataformas verticales infinitas.
> Este archivo define cómo trabajar en el proyecto. No es documentación de diseño ni reemplaza al GDD.
> Estado del proyecto verificado el **26/10/2026**.

---

## 0. Contexto verificado del proyecto

| Aspecto | Estado real |
|---|---|
| Documento de requisitos | `GDD_ASCENSO_v1.0.md` (raíz del proyecto) |
| Carpeta `docs/` | No existe |
| `AGENTS.md` previo | No existía |
| Entrada del juego | `index.html` |
| Motor | Phaser 3.90.0 cargado por CDN |
| Lenguaje | JavaScript clásico: IIFE + namespace global `window.Ascenso` |
| Build | No hay `package.json`, bundler ni transpilador |
| Pruebas | No hay framework, runner ni scripts de prueba |
| Git | No hay repositorio y el comando `git` no está instalado |
| Recursos | No hay imágenes, sprites ni audio externo; el audio es procedural |
| Resolución de diseño | 720 × 1280 (9:16) |
| Herramientas locales | Python 3.14.7 disponible; Node.js y npm no disponibles |

### 0.1 Mapa real de archivos

```text
WORKSHOPDT/
├── GDD_ASCENSO_v1.0.md   Documento de diseño y requisitos (v1.0)
├── index.html            Entrada, estilos, loader y orden de scripts
└── src/
    ├── audio.js          Audio procedural (WebAudio)
    ├── config.js         Constantes, fases, paletas, skins, ajustes
    └── utils.js          RNG con semilla, persistencia, récords
```

### 0.2 Módulos declarados por `index.html` que aún no existen

`index.html` carga estos archivos, pero **ninguno está presente**:

- `src/art.js`
- `src/background.js`
- `src/ui.js`
- `src/scenes/BootScene.js`
- `src/scenes/MenuScene.js`
- `src/scenes/GameScene.js`
- `src/scenes/UIScene.js`
- `src/main.js`

Consecuencia: **el juego no arranca**. No afirmes que una pantalla, mecánica o escena está implementada solo porque `index.html` la mencione.

### 0.3 Brechas conocidas entre documentación, código y estado real

| Brecha | Evidencia | Consecuencia |
|---|---|---|
| El juego no inicia | `index.html` referencia 8 archivos ausentes | Verificar antes de afirmar funcionamiento |
| `npm run vendor` no existe | Comentario en `index.html`; no hay `package.json` | No ejecutar ese comando |
| Motor distinto al GDD | GDD §2 recomienda Godot 4.3; el código es Phaser 3.90 | Phaser es la base técnica decidida; el GDD manda en diseño |
| "Top 5" no implementado | GDD §2 lo menciona; `A.DEFAULT_RECORDS` no lo contiene | No inventar su estructura sin consultar |
| Catch-up con "+40 px/s" sin constante | GDD §11.2; `A.CAMERA` no la define | Tratarlo como pendiente |
| Margen de alcance del 15 % sin constante | GDD §10.3; no hay valor en `src/config.js` | Tratarlo como pendiente |
| Timestep 120 Hz, pausa al perder foco y gamepad no verificables | GDD §2 y §7; falta `main.js` | No afirmar que están implementados |
| Sprites del GDD ausentes | GDD §17 describe 56 × 56 y frames; no hay assets | El arte es procedural por decisión del proyecto |
| Título "Pies de pluma" sin persistencia | GDD §13.2; no hay dato asociado | Tratarlo como pendiente |

---

## 1. Alcance y contexto

Aplicar estas instrucciones **al inicio de cada consulta** y **antes de modificar el proyecto**.

- Responder en **español**.
- Adaptar la profundidad del análisis a la complejidad de la solicitud: breve para consultas simples, detallada para cambios de comportamiento.
- Identificar las **rutas reales** antes de citar:
  - Documentación: `GDD_ASCENSO_v1.0.md` en la raíz.
  - No existe `docs/`. Si se crea en el futuro, revisarla antes del GDD.
  - Documento principal de requisitos: `GDD_ASCENSO_v1.0.md`.
- Revisar la información vigente del repositorio. **No depender exclusivamente de resúmenes de conversaciones anteriores**, incluidos los de este archivo: si el proyecto cambió, verificar de nuevo.
- Preservar el trabajo existente del usuario. No borrar, mover, renombrar ni sobrescribir archivos sin necesidad y sin consultar cuando haya riesgo.
- Si un dato del proyecto no coincide con este archivo, **prevalece el estado real verificado**; informar la diferencia.

---

## 2. Revisar la documentación y el requisito

Antes de proponer o implementar una solución:

1. **Identificar el requisito** solicitado y su comportamiento esperado.
2. **Revisar la documentación** disponible:
   - `GDD_ASCENSO_v1.0.md`.
   - `docs/`, si llega a existir.
   - Cualquier otro documento de diseño, balance o QA que se agregue.
3. **Consultar las secciones pertinentes del GDD**, que es el documento principal de requisitos de este videojuego. Mapa rápido:

| Tema | Sección del GDD |
|---|---|
| Reglas del juego | §8 |
| Estados y física del jugador | §9 |
| Tipos, probabilidades y generación de plataformas | §10 |
| Scroll, cámara, línea de muerte y rescate | §11 |
| Dificultad y balance | §12 |
| Puntuación y desbloqueos | §13 |
| Feedback y *juice* | §14 |
| Ilusiones visuales | §15 |
| UI, pantallas, tutorial y accesibilidad | §16 |
| Dirección de arte | §17 |
| Audio | §18 |
| Estados y flujo de escenas | §20 |
| Alcance del MVP | §22.1 |
| Constantes de ajuste | §24 |
| Checklist de QA | §25 |
| Glosario | §26 |
| Decisiones abiertas | §27 |

4. **Clasificar el requisito**:
   - **Definido:** la documentación describe suficientemente el comportamiento.
   - **Parcialmente definido:** está contemplado, pero faltan detalles necesarios.
   - **No definido:** no aparece en la documentación revisada.
   - **En contradicción:** la solicitud difiere de una regla documentada.
   - **No aplica:** es una consulta técnica o documental sin una regla funcional asociada.
5. **Citar el documento y la sección** que respaldan el análisis:
   - Markdown: `GDD_ASCENSO_v1.0.md §10.2 «Probabilidades por fase»`.
   - PDF, si se agrega: indicar página y distinguir numeración del archivo y numeración impresa cuando difieran.
   - Código: ruta y símbolo, por ejemplo `src/config.js`, `A.PHASES`.
6. **Diferenciar** claramente:
   - Regla documentada.
   - Propuesta del agente.
   - Supuesto necesario.
   - Decisión pendiente del usuario.

### 2.1 Reglas cerradas y decisiones abiertas

El GDD contiene reglas que también aparecen como preguntas en §27 «Decisiones abiertas». No las trates como completamente definidas ni elijas una interpretación por tu cuenta.

| Tema | Regla aparente | Referencia | Tratamiento |
|---|---|---|---|
| Anticipación del rebote | 0,05 s | §9.1, §9.2 | También abierta en §27.1 → **parcialmente definido** |
| Gracia tras la línea de muerte | 0,20 s | §9.2, §11.3 | También abierta en §27.2 → **parcialmente definido** |
| Ranking del modo calma | Separado | §12.3, §16.4 | También abierta en §27.4 → **parcialmente definido** |
| Tope de scroll | 170 px/s | §11.1, §11.3 | También abierta en §27.5 → **parcialmente definido** |
| Revivir / revancha | No contemplado en las reglas | §27.6 | **No definido** |
| Motor | Godot 4.3 | §2 | **Decisión del proyecto:** Phaser 3.90; no migrar sin pedido explícito |
| Top 5 local | Persistencia con top 5 | §2 | **Parcialmente definido**: no está en `src/utils.js` |
| Sprites | Personaje 56 × 56 con frames | §17 | **No hay archivos**; arte procedural por decisión |

Si una tarea depende de una de estas decisiones, preguntar antes de modificar el comportamiento afectado.

### 2.2 Límites de esta revisión

- Si la documentación **no existe o no puede leerse**, indicarlo claramente y **no afirmar** que el requisito está definido o ausente.
- Un requisito **no documentado** puede ser una ampliación válida: identificarlo como tal y consultar las decisiones necesarias.
- **No modificar el GDD ni la documentación de requisitos** para justificar una implementación sin una solicitud explícita del usuario.
- `src/config.js` contiene valores derivados del GDD, pero **no es la fuente de verdad**. Si el código y el GDD difieren, reportarlo y consultar; por defecto, prevalece el GDD.

---

## 3. Analizar conflictos e impacto

Antes de modificar una funcionalidad:

- Revisar tanto las **reglas documentadas** como la **implementación existente**.
- Identificar dependencias, restricciones compartidas, excepciones, prioridades entre comportamientos y posibles regresiones.
- Analizar interacciones con otras funcionalidades.
- Comparar el comportamiento actual con la documentación. **No asumir que el código existente es correcto.**
- Detectar contradicciones internas del GDD, incluidas diferencias entre texto, diagramas ASCII, tablas e imágenes:
  - Diagrama de línea de muerte y rescate: §11.3.
  - Curva de dificultad: §12.1.
  - Diagrama de flujo de escenas: §20.
- Comunicar los conflictos relevantes **antes** de modificar la parte afectada.

### 3.1 Checklist específica del dominio

| Área | Qué verificar | Referencia |
|---|---|---|
| Fases y scroll | Umbrales 0/10/25/50/80/120/180, velocidades 70–170, interpolación de 3 s, aviso 0,8 s antes | §11.1, `A.PHASES` |
| Cámara | Catch-up a 1,8×, zona superior del 25 %, retorno al 35 %, línea de muerte a 80 px, gracia de 0,20 s | §11.2–11.3, `A.CAMERA` |
| Generación | 4–6 plataformas por encima, 1–2 por banda, una segura cada 2 bandas, separación mínima 60/40, alcance validado, primeras 5 de tutorial, semilla guardada | §10.3 |
| Tipos de plataforma | Probabilidades por fase, espejismo nunca único ni antes del salto 40, muelle con plataforma alcanzable arriba, frágil que no se rompe antes de abandonarla, móvil que se detiene al aterrizar | §10.1–10.2 |
| Física y colisión | One-way, colisión barrida sin *tunneling*, asistencia de borde de 8 px, penetración de 6 px, paredes laterales, ausencia de salto manual | §8, §9.2, §9.4 |
| Puntuación | +1 por plataforma nueva, perfecto con centro del 30 % y +2, hitos de combo 5/10/20, multiplicador visual con tope ×2,5, récords | §13.1 |
| Persistencia | Ajustes, récords, semilla diaria, modo calma, top 5 pendiente | §2, §13, §16.4, `A.Store` |
| Escenas | Boot, menú, tutorial, partida, pausa, muerte, game over | §20 |
| Controles | Teclado, táctil, gamepad, buffer de dirección de 0,12 s, pausa al perder foco | §7 |
| UI | Posiciones del HUD, desvanecido, *safe areas*, pantallas | §16 |
| Accesibilidad | Alto contraste, movimiento reducido, modo calma, sin dependencia del color, sin audio obligatorio | §16.4 |
| Feedback | *Shake* máximo 8 px, flashes máximo 3 Hz, *hitstop*, *slow-mo* de muerte | §14 |
| Ilusiones | Solo cosméticas o telegrafiadas; **el hitbox nunca miente** | §15 |
| Audio | Cuatro capas, BPM 90–140, *riser* de 0,8 s, *ducking*, funcionamiento sin audio | §18 |
| Resolución | 720 × 1280, escalado, respeto de *safe areas* | §2, `index.html` |
| QA | Checklist completa antes de cerrar un cambio | §25 |

---

## 4. Preguntar antes de avanzar ante dudas

Cuando una duda afecte el alcance, las reglas, el comportamiento, la compatibilidad o la implementación:

- Utilizar la herramienta de preguntas disponible en el entorno, **`question`**.
- Formular preguntas concretas.
- Explicar **qué decisión falta** y **por qué importa**.
- Ofrecer opciones cuando faciliten la decisión.
- Incluir una recomendación fundamentada cuando corresponda.
- **Esperar la respuesta** antes de adoptar una decisión que cambie el comportamiento solicitado.
- **No inventar reglas** para resolver omisiones o contradicciones.
- Continuar con tareas independientes cuyo alcance esté claro.

Si `question` no está disponible, preguntar directamente en la conversación y esperar la aclaración necesaria.

---

## 5. Utilizar POO y patrones de diseño

- Modelar entidades y comportamientos del dominio con programación orientada a objetos, respetando **JavaScript clásico, Phaser 3 y la arquitectura existente**.
- Mantener el patrón del proyecto:
  - Cada archivo se envuelve en una IIFE.
  - Se expone lo necesario en `window.Ascenso` (`A`).
  - No se usan `import`/`export` ni módulos ES sin autorización, porque `index.html` carga scripts clásicos y debe seguir funcionando en local.
  - No introducir TypeScript, frameworks, bundlers ni transpiladores sin autorización.
- Encapsular estado y reglas. Evitar variables globales sueltas fuera de `A`.
- Mantener responsabilidades claras, alta cohesión y bajo acoplamiento.
- Favorecer composición cuando evite jerarquías de herencia innecesarias.
- Separar lógica de negocio, presentación y entrada del usuario.
- Usar los valores de `src/config.js` (`A.PLAYER`, `A.CAMERA`, `A.PHASES`, etc.) en lugar de números mágicos.
- Comentarios y textos de interfaz en español; identificadores siguiendo el estilo existente del proyecto.
- Respetar el orden de carga de `index.html`: `config → utils → audio → art → background → ui → escenas → main`. Si se agrega un archivo, insertarlo en el orden correcto.
- Utilizar patrones de diseño cuando resuelvan una necesidad concreta y **explicar brevemente** la elección y su utilidad.
- Evitar abstracciones innecesarias y refactorizaciones ajenas al requisito.

Patrones orientativos para este proyecto:

| Necesidad | Patrón posible |
|---|---|
| Estados del jugador (`IDLE`, `ASCEND`, `FALL`, `LAND`, `DEAD`) y flujo de escenas | **State** |
| Comportamiento variable por tipo de plataforma, fase o causa de muerte | **Strategy** |
| Creación de bandas, plataformas y entidades con variantes | **Factory** |
| Actualización de HUD, audio y feedback ante eventos del juego | **Observer** |
| Reciclado de plataformas que quedan 200 px bajo la línea de muerte | **Object Pool** |
| Fachada de audio procedural (`A.audio`) | **Facade** |

No imponer estos patrones cuando no aporten valor.

---

## 6. Reutilizar código existente

Antes de crear clases, componentes, servicios o utilidades:

1. Buscar implementaciones relacionadas en `src/`.
2. Revisar sus contratos, comportamiento y consumidores.
3. Priorizar su reutilización o extensión cuando sean compatibles.
4. Evitar duplicar lógica de negocio.
5. Extraer lógica común cuando exista una necesidad real, preservando el comportamiento de sus consumidores.
6. Respetar las convenciones de estructura, nombres y estilo del proyecto.

Puntos de reutilización verificados:

| Necesidad | Reutilizar |
|---|---|
| Matemáticas y utilidades | `A.clamp`, `A.lerp`, `A.approach`, `A.smoothstep`, `A.rand`, `A.choice` |
| RNG reproducible | `A.rng(seed)` (Mulberry32) |
| Persistencia local | `A.Store`, `A.loadSettings`, `A.saveSettings`, `A.loadRecords`, `A.saveRecords` |
| Fusión de valores por defecto | `A.mergeDefaults` |
| Récords y modos | `A.commitRun`, `A.bestForMode`, `A.skinFor`, `A.trailFor` |
| Fechas y semilla diaria | `A.dateKey`, `A.seedFromDate`, `A.dateLabel` |
| Formato | `A.formatInt`, `A.pad2` |
| Vibración | `A.vibrate` |
| Audio | `A.audio` (`AudioEngine`) |
| Constantes de diseño | `A.PLAYER`, `A.CAMERA`, `A.PLATFORMS`, `A.SCORE`, `A.PHASES`, `A.PALETTES`, `A.SKINS`, `A.TRAILS`, `A.COLORS` |
| Ajustes y récords por defecto | `A.DEFAULT_SETTINGS`, `A.DEFAULT_RECORDS` |

No reimplementes `localStorage`, el RNG, el sistema de récords ni el motor de audio si las utilidades existentes cubren la necesidad.

Si no existe código reutilizable, indicarlo y diseñar una solución coherente con el proyecto.

Repetir esta revisión en cada tarea; **no asumir que la estructura permanece igual entre consultas**.

---

## 7. Consultar sobre sprites y recursos gráficos

Estado actual: **no hay archivos de imagen, sprites, spritesheets ni audio externo**. El arte previsto es procedural, en la línea de `src/art.js` (aún ausente) y `src/audio.js`.

Cuando una tarea requiera sprites u otros recursos gráficos:

1. Revisar los recursos disponibles y sus convenciones de uso.
2. Utilizar la herramienta `question` **antes de implementar la parte visual** para confirmar si el usuario desea:
   - **Generar un mockup:** representación provisional para desarrollar y validar la funcionalidad.
   - **Utilizar un recurso existente:** solicitar el **nombre exacto** del recurso.
3. Si el recurso indicado no puede localizarse, pedir su ruta o el archivo correspondiente.

Además:

- **No asumir** qué sprite utilizar ni **inventar nombres o rutas** de archivos.
- No crear una carpeta `assets/` ni una convención de nombres sin acordarla.
- Si se elige un mockup, aclarar que es **provisional** y confirmar el alcance visual necesario.
- Si se elige un recurso existente, comprobar su disponibilidad y compatibilidad con el uso previsto.
- Consultar detalles como spritesheet, fotogramas, animaciones, escala y punto de anclaje cuando sean necesarios y no estén documentados.
- El GDD §17 describe un personaje de 56 × 56 px con 8 frames de rebote, 4 de caída y 2 de *squash*. Es una **referencia de diseño**, no un archivo disponible.
- Si falta el archivo, solicitarlo antes de avanzar con la parte que depende de él.
- Continuar con la lógica independiente de los recursos gráficos mientras se resuelve la consulta.

---

## 8. Definir y preparar el flujo Git

### 8.1 Comprobación obligatoria

Antes de implementar, comprobar el estado real:

```powershell
git rev-parse --is-inside-work-tree
git status --short
git branch --show-current
git remote -v
git log --oneline -10
```

**Estado verificado el 26/10/2026:** `git` no está instalado y la carpeta no es un repositorio. No hay rama, remoto ni estrategia documentada. El GDD solo menciona Git como herramienta en §22.4.

Si `git` no está disponible o no hay repositorio:

- Informar la situación.
- **No ejecutar `git init`**, no configurar remotos y no instalar Git sin autorización explícita.
- Consultar cómo proceder: trabajar sin Git, instalar Git, inicializar un repositorio o usar un repositorio existente.
- No simular ramas, commits ni push, ni afirmar que se realizaron.

### 8.2 Si existe repositorio

- Revisar estado, rama actual, remotos y estrategia documentada.
- Respetar el flujo existente.
- Si se usa GitFlow, seguir sus ramas y convenciones para features, fixes, hotfixes y releases.
- Si no hay estrategia definida, preguntar:
  - Rama base.
  - Convención de nombres.
  - Remoto.
- **No asumir que `develop` existe** ni imponer GitFlow completo sin acordarlo.

Crear una rama de trabajo antes de modificar código, con un nombre descriptivo:

```text
feature/<descripcion>
fix/<descripcion>
docs/<descripcion>
refactor/<descripcion>
test/<descripcion>
```

Si ya existe una rama apropiada para la tarea, verificar si corresponde continuar en ella.

**No implementar directamente sobre `main`, `master` ni `develop`**, salvo que el usuario indique explícitamente otra estrategia.

### 8.3 Cambios previos del usuario

- Preservarlos.
- Identificar si interfieren con la tarea.
- Consultar antes de realizar operaciones que puedan moverlos, descartarlos o mezclarlos con otros cambios.
- No usar `git stash`, `git reset`, `git checkout --`, `git clean` ni `git restore` destructivos sin autorización.

---

## 9. Implementar y verificar

- Definir criterios de aceptación a partir del requisito, el GDD y las aclaraciones del usuario.
- Realizar cambios enfocados en el alcance acordado.
- Ejecutar las comprobaciones disponibles y pertinentes al cambio.
- Verificar las interacciones identificadas durante el análisis de impacto.
- Agregar pruebas cuando aporten valor para validar reglas o prevenir regresiones.
- Evitar pruebas que solo repliquen la implementación.
- Informar qué se verificó y qué quedó pendiente.
- **No afirmar resultados que no fueron comprobados.**

### 9.1 Verificación disponible en este proyecto

No hay Node.js, npm, `package.json`, linter ni framework de pruebas. **No inventar comandos** como `npm test`, `npm run build` o `npm run vendor`.

Opciones reales:

1. **Servidor local con Python** (Python 3.14.7 verificado):

   ```powershell
   python -m http.server 8000
   ```

   Luego abrir `http://localhost:8000/` en el navegador.

2. **Abrir `index.html` directamente** en el navegador. Los scripts son clásicos, pero Phaser se carga desde CDN y requiere conexión a internet.

3. **Consola del navegador:** revisar errores de carga, errores de JavaScript y el estado de `window.Ascenso`.

4. **Panel Network:** comprobar que Phaser y los scripts locales cargan correctamente. Tener en cuenta que los 8 módulos ausentes generarán errores.

5. **Checklist manual del GDD §25** para los comportamientos que se puedan probar.

6. **Contraste de constantes** entre `src/config.js` y el GDD:
   - `A.PLAYER` frente a §9.2 y §24.
   - `A.CAMERA` frente a §11 y §24.
   - `A.PHASES` frente a §10.2 y §11.1.
   - `A.PALETTES` frente a §17.

Si se incorpora un framework de pruebas, pedir autorización, documentar el comando exacto y no presentar pruebas no ejecutadas como exitosas.

---

## 10. Solicitar confirmación y realizar commit y push

Cuando termine la implementación:

1. Presentar al usuario:
   - Resumen del comportamiento implementado.
   - Archivos afectados.
   - Verificaciones realizadas y sus resultados.
   - Pendientes o limitaciones reales.
   - Rama de trabajo utilizada, si existe.
2. Utilizar `question` para solicitar:
   - Confirmación de que la implementación es aceptada.
   - Autorización explícita para realizar commit y push.
3. Esperar la respuesta.
4. Si el usuario solicita ajustes:
   - Realizarlos.
   - Repetir las verificaciones afectadas.
   - Volver a solicitar confirmación.

Después de recibir autorización:

1. Revisar `git status`, `git diff`, `git diff --staged` y `git log --oneline -10`.
2. Revisar los archivos nuevos que se incluirán.
3. Preparar **únicamente** los cambios de la tarea, sin incluir cambios ajenos ni secretos. Preferir `git add <ruta>` sobre `git add .`.
4. Crear un commit descriptivo. Si el repositorio no documenta una convención, usar mensajes en español con prefijo (`feat:`, `fix:`, `docs:`, `refactor:`, `test:`) y confirmar la preferencia con el usuario.
5. Hacer push de la rama al remoto acordado, configurando su seguimiento cuando corresponda:
   ```powershell
   git push -u origin <rama>
   ```
6. Informar:
   - Nombre de la rama.
   - Identificador y mensaje del commit.
   - Resultado del push.

Reglas:

- **No realizar commit ni push sin autorización explícita del usuario.**
- No hacer force push.
- No omitir hooks (`--no-verify`).
- No modificar la configuración de Git.
- No fusionar ramas sin una solicitud expresa.
- Si un hook o comprobación falla, resolver el problema antes de continuar. No saltarlo.
- Si commit o push falla, informar el resultado sin afirmar que la operación se completó.
- La creación de un pull request y la integración en la rama base requieren autorización adicional.
- Si no hay Git disponible, **no hay commit ni push posibles**: informarlo y pedir decisión. No inventar hashes ni resultados.

---

## 11. Comunicar el análisis y los resultados

Antes de implementar, presentar un análisis breve con los puntos pertinentes:

### Análisis previo

- **Requisito:** comportamiento solicitado.
- **Documentación:** estado y referencias (`GDD_ASCENSO_v1.0.md §…`).
- **Conflictos e impacto:** funcionalidades y reglas afectadas.
- **Reutilización:** código existente aprovechable.
- **Diseño:** enfoque POO y patrones pertinentes.
- **Recursos gráficos:** recursos confirmados o consulta pendiente.
- **Git:** rama base y rama de trabajo, o ausencia de repositorio.
- **Dudas:** decisiones que requieren respuesta del usuario.

Para consultas simples, reducir el formato a los puntos aplicables.

Al finalizar, resumir:

- Cambios y archivos afectados.
- Decisiones relevantes.
- Verificaciones ejecutadas y resultados.
- Pendientes reales.
- Confirmación solicitada al usuario.
- Resultado de commit y push, únicamente si fueron autorizados y ejecutados.

**No presentar propuestas como funcionalidades implementadas.**

---

## 12. Flujo esperado para futuras implementaciones

```text
revisar documentación
        ↓
analizar impacto
        ↓
aclarar dudas
        ↓
confirmar recursos gráficos cuando corresponda
        ↓
definir y crear rama (si hay Git)
        ↓
diseñar y reutilizar
        ↓
implementar
        ↓
verificar
        ↓
solicitar confirmación
        ↓
commit
        ↓
push
```

---

## 13. Decisiones registradas del proyecto

| Fecha | Decisión | Alcance |
|---|---|---|
| 26/10/2026 | **Phaser 3 + JavaScript clásico** es la base técnica; el GDD manda en diseño y reglas | No migrar a Godot sin pedido explícito |
| 26/10/2026 | Los 8 módulos ausentes de `index.html` están **pendientes de implementar** | No asumir que existen ni que el juego funciona |
| 26/10/2026 | Flujo Git **condicional**: se aplica solo si hay repositorio y Git disponible; no inicializar automáticamente | Informar y consultar antes de instalar o inicializar |
| 26/10/2026 | Arte **procedural por defecto**; para sprites, consultar mockup vs recurso existente | No inventar assets ni rutas |

Estas decisiones no modifican el GDD. Si el usuario cambia alguna, actualizar este archivo y dejar constancia.

---

## 14. Prohibiciones rápidas y mantenimiento

No hacer:

- Afirmar que una funcionalidad está implementada sin verificarla.
- Afirmar que el juego funciona cuando faltan módulos.
- Modificar el GDD para justificar una implementación.
- Inventar reglas, secciones, sprites, rutas, tecnologías o comandos.
- Instalar dependencias, Git o herramientas sin autorización.
- Ejecutar `git init`, `git push --force`, `--no-verify` o cambios de configuración sin autorización.
- Hacer commit o push sin autorización explícita.
- Refactorizar fuera del alcance o duplicar lógica existente.
- Borrar, mover o sobrescribir trabajo del usuario sin consultar.
- Presentar supuestos como reglas documentadas.

Mantener este archivo actualizado cuando cambie el estado real del proyecto: nuevos documentos, creación de `docs/`, inicialización de Git, incorporación de assets, framework de pruebas o cambios de alcance. Verificar las rutas antes de citarlas.

---

## 15. Checklist rápido antes de responder

1. ¿Leí el GDD y cité la sección aplicable?
2. ¿Clasifiqué el requisito (definido, parcial, no definido, contradictorio, no aplica)?
3. ¿Revisé el código existente y sus consumidores?
4. ¿Identifiqué conflictos e impacto?
5. ¿Pregunté lo que no puedo resolver con la documentación?
6. ¿Confirmé los recursos gráficos si la tarea es visual?
7. ¿Verifiqué si hay Git y qué rama corresponde?
8. ¿Informé qué verifiqué y qué quedó pendiente?
