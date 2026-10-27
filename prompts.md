# Prompts

Aquí van **todos los prompts que lanzaste** para hacer el ejercicio, en el orden en que los
lanzaste, con el modelo y la herramienta de cada uno.

Esto no es papeleo. Lo que se revisa es **cómo pediste las cosas**, no solo lo que salió: un
resultado flojo con un prompt bueno y un resultado flojo con un prompt vago necesitan feedback
distinto, y sin este archivo no se distinguen.

## Cómo rellenarlo

- Un apartado `## Prompt N` por cada prompt.
- **Pega el prompt tal cual lo lanzaste**, dentro del bloque de código, aunque ocupe diez líneas
  y aunque tenga faltas. No lo reescribas para que quede bien: el que arreglaste mentalmente
  después no es el que lanzaste.
- Incluye también los que **no funcionaron**. Suelen ser los más útiles de leer.
- `Modelo` y `Herramienta` en todos. Si cambiaste de una a otra a mitad, se nota aquí.

Borra el ejemplo de abajo cuando escribas el primero.

---

## Prompt 1

**Modelo:** Sonnet 5.5
**Herramienta:** Claude Code
**Hora:** 2026-10-20 22:34:59 (reloj de 45 min iniciado)

```
Actúa como un ingeniero de software senior que realiza una auditoría técnica de un sistema heredado.
Necesito auditar exclusivamente la feature de comentarios de este blog: crear, editar, borrar y listar comentarios, considerando la API, la base de datos y la interfaz web.
Primero, inspecciona la estructura del repositorio y localiza los archivos relacionados con comentarios. No audites posts, autores, categorías ni etiquetas, salvo que necesites comprobar una dependencia directa de comentarios.
Busca posibles hallazgos en el código existente. Para cada hallazgo, proporciona:

1. Una descripción concreta en una frase.
2. El archivo y la línea exacta que respalda la afirmación.
3. El efecto observable y quién resultaría afectado.
4. El número exacto de sitios de código que sería necesario modificar, solo si puedes contarlos y justificarlo.
5. Si el problema se puede revertir con Git sin dejar efectos persistentes y por qué.
6. Qué otro hallazgo debe precederlo o si no bloquea a ninguno.

No modifiques el código, no corrijas errores, no alteres la base de datos y no ejecutes operaciones que cambien datos. En esta primera fase, inspecciona el código y presenta los hallazgos candidatos con sus evidencias.
Distingue expresamente entre hechos comprobados, hipótesis y afirmaciones pendientes de verificación. No inventes líneas, impactos, cantidades ni dependencias. Si no puedes demostrar una afirmación, indícalo.
No intentes auditar todo el sistema ni producir una lista extensa por cumplir una cuota. Prioriza la evidencia y el alcance limitado de la feature. Espera mi siguiente instrucción antes de modificar cualquier archivo.
Registra este prompt literalmente en prompts.md junto con el modelo y la herramienta utilizados.
```

**Qué salió:** funcionó a la primera: el agente localizó los archivos de comentarios, devolvió 11 candidatos con sus seis datos y los separó en hechos comprobados, hipótesis y pendientes de verificar. Respetó las restricciones: no modificó código ni datos (las pruebas en vivo fueron solo `GET` y cuatro `POST` que el servidor rechaza antes de insertar; la tabla siguió en 9 filas) y no escribió todavía el archivo de la auditoría, a la espera de la siguiente instrucción.

## Prompt 2

**Modelo:** Sonnet 5.5
**Herramienta:** Claude Code
**Hora:** 2026-10-20 22:41:14

```
Vamos a continuar con la auditoría, pero necesito verificar personalmente los hallazgos antes de priorizarlos.
Abre en el repositorio los archivos y líneas exactas que respaldan C1, C2 y C3. Para cada uno:

1. Muéstrame el fragmento real del archivo, con suficiente contexto para entenderlo.
2. Explícame qué demuestra directamente el código y qué parte sigue siendo una hipótesis.
3. Indica qué evidencia adicional sería necesaria para confirmar el impacto.
4. No modifiques código, datos ni configuración, y no ejecutes pruebas que puedan borrar o insertar información.
5. No marques un hallazgo como verificado por mí: yo decidiré cuándo lo he comprobado abriendo el archivo.

Después nos detendremos para registrar mis comprobaciones reales. No redactes todavía la auditoría definitiva ni inventes los valores de la Parte B. Conserva en prompts.md el texto literal de cada prompt utilizado, junto con el modelo y la herramienta reales.
```

**Qué salió:** el agente releyó los archivos y mostró los fragmentos reales de C1, C2 y C3, separó lo que demuestra el código de lo que sigue siendo hipótesis, indicó qué evidencia faltaría, abrió `routes.ts` en el panel de archivos y no marcó nada como verificado por el usuario. No modificó código, datos ni configuración.

## Prompt 3

**Modelo:** Sonnet 5.5
**Herramienta:** Claude Code
**Hora:** 2026-10-20 22:45:22

```
git status --short
```

**Qué salió:** el prompt era solo un comando, sin instrucción; el agente lo interpretó como una petición de ejecutarlo y mostrar el resultado. Salieron 6 archivos de `backend/.adonisjs/` modificados (ruido de saltos de línea, sin cambios reales) y `prompts.md` (el registro de estos prompts). No hay ningún archivo de la auditoría todavía.

## Prompt 4

**Modelo:** Sonnet 5.5
**Herramienta:** Claude Code
**Hora:** 2026-10-20 22:47:12

```
No hagas stash ni restaures archivos todavía. Primero quiero preservar el estado actual del repositorio.

Confirma que los seis archivos de backend/.adonisjs/ no tienen diferencias reales de contenido y que sus cambios son únicamente de formato de fin de línea. No descartes ni restaures cambios sin mi autorización.

Después, continuemos con la verificación de C1, C2 y C3, mostrándome los fragmentos directamente desde los archivos para que yo pueda abrirlos y comprobarlos. No marques como verificados por mí los hallazgos que todavía no he revisado.

Mantén el registro literal de los prompts en prompts.md, sin inventar ni reconstruir los prompts anteriores.
```

**Qué salió:** el agente no hizo stash ni restauró nada. Comprobó los seis archivos de `backend/.adonisjs/` con tres medidas (comparación byte a byte con `cmp`, hash de git de HEAD, índice y copia de trabajo, y recuento de retornos de carro) y no hay ninguna diferencia de contenido. Pero la premisa del prompt era inexacta: tampoco son diferencias de fin de línea, porque no hay ningún retorno de carro en ninguno de los dos lados; git los marca como modificados por la información de fecha guardada en el índice, ya que el servidor reescribió los archivos con el mismo contenido. Esto corrige la nota del Prompt 3, que los llamaba «ruido de saltos de línea». Después mostró los fragmentos de C1, C2 y C3 leídos directamente con `awk` desde los archivos y abrió `routes.ts` en el panel; no marcó nada como verificado por el usuario.

## Prompt 5

**Modelo:** Sonnet 5.5
**Herramienta:** Claude Code
**Hora:** 2026-10-20 22:55:33

```
He revisado personalmente los fragmentos de código de C1, C2 y C3 en el editor.

Registra como revisados únicamente los hallazgos cuyos fragmentos efectivamente abrí. Distingue la evidencia confirmada en el código de los impactos que siguen pendientes de verificación, especialmente los relacionados con producción.

No sigas investigando los archivos de backend/.adonisjs/: los hashes y la comparación byte a byte indican que su contenido coincide con Git. No los restaures ni los incluyas en el commit.

Ahora prepara docs/auditoria/comentarios.md con los hallazgos respaldados por evidencia, la tabla de seis columnas y las tres líneas obligatorias de la Parte B. Mantén en blanco las casillas que no hayas podido completar; no inventes datos ni afirmes que he verificado algo que no he revisado.

Antes de terminar, muéstrame el contenido del archivo para revisarlo. No hagas commit ni push todavía.
```

**Qué salió:** el agente creó `docs/auditoria/comentarios.md` con una tabla de 9 filas y seis casillas, una sección de qué está confirmado y qué no, las propuestas sin medir fuera de la tabla y la Parte B. Registró como revisados por el usuario solo C1, C2 y C3, según su declaración. Dejó en blanco las casillas que no pudo contar (sitios de C11, C5 y C2b) y las líneas 2 y 3 de la Parte B, porque dependen del juicio del usuario. No tocó los archivos de `backend/.adonisjs/` y no hizo commit ni push.

## Prompt 6

**Modelo:** Sonnet 5.5
**Herramienta:** Claude Code
**Hora:** 2026-10-20 23:02:25

```
2. Prioridad que no supe decidir: No supe decidir si priorizar C1 o C11: C1 expone correos en las respuestas de la API local y urge limitar los datos devueltos, pero C11 plantea la necesidad de tener pruebas de caracterización antes de modificar el comportamiento heredado.
3. Afirmación no verificada: El agente planteó que podría existir un panel de moderación fuera del repositorio, a partir de una referencia en la semilla de datos, pero no pude comprobar si ese panel existe ni si está conectado a esta API.
```

**Qué salió:** el mensaje era solo el texto de las líneas 2 y 3 de la Parte B, sin instrucción; el agente lo interpretó como el texto que debía quedar en `docs/auditoria/comentarios.md` y lo escribió tal cual, sin cambiar palabras, en lugar de los dos marcadores «pendiente». No hizo commit ni push.

## Prompt 7

**Modelo:** Sonnet 5.5
**Herramienta:** Claude Code
**Hora:** 2026-10-20 23:06:09

```
Haz los últimos ajustes en docs/auditoria/comentarios.md, sin modificar código ni tocar otros archivos:
1. Conserva las seis columnas obligatorias y resume el texto de las celdas para que la tabla sea corta y fácil de revisar en una pantalla.
2. Mantén fuera de la tabla C8, C9, C10 y las propuestas sin medir, junto con el experimento necesario para verificarlas.
3. Elimina la matriz adicional «Qué está confirmado y qué no» si repite información de la tabla. Conserva las incertidumbres relevantes en las filas correspondientes.
4. Revisa que el orden sea coherente con las dependencias y, dentro de cada nivel de precedencia, con el daño dividido por el radio. No presentes el orden como definitivo si la evidencia no permite resolverlo.
5. Conserva las tres líneas de la Parte B. En la segunda utiliza el conflicto entre C1 y C11 descrito en este mensaje; en la tercera, la incertidumbre sobre el panel externo de moderación.
6. Mantén la primera línea en 11 hallazgos devueltos y 3 comprobados solo si esos son efectivamente los números del registro y abrí personalmente los tres archivos. No marques como comprobados por mí los demás.
7. No rellenes casillas que continúan sin evidencia, no inventes información y no hagas commit ni push. Muéstrame el contenido final de comentarios.md para la última revisión.
```

**Qué salió:** el agente acortó las celdas de la tabla (seguía con seis columnas y 9 filas), movió a las filas las incertidumbres que importaban (producción, panel externo, correo correcto de C3, estados reales) y eliminó la matriz «Qué está confirmado y qué no». Mantuvo fuera de la tabla C8, C9, C10 y el tamaño/anti-spam, cada uno con su experimento. Reordenó por niveles de precedencia y dejó dicho que el orden es una propuesta, porque el daño no está cuantificado. Conservó las tres líneas de la Parte B; la primera sigue en 11 y 3 porque el registro del Prompt 1 devolvió 11 candidatos y el usuario declaró haber abierto C1, C2 y C3. Dejó en blanco los sitios de C11, C5 y C2b. No hizo commit ni push.

## Prompt 8

**Modelo:** Sonnet 5.5
**Herramienta:** Claude Code
**Hora:** 2026-10-20 23:10:05

```
si apruebo
```

**Qué salió:** respuesta a la pregunta de si aprobaba el contenido final y si quería commit y push. El agente lo interpretó como aprobación de ambos: hizo commit de `docs/auditoria/comentarios.md` y `prompts.md` con el mensaje de la guía del curso y push de `auditoria-rm`. No abrió el PR.

## Prompt 9

**Modelo:** Sonnet 5.5
**Herramienta:** Claude Code
**Hora:** 2026-10-20 23:11:42

```
pasemos al PR contra s6/start
```

**Qué salió:** el agente abrió el PR 9 en `LIDR-academy-temp/blog-legacy-ai4devs-202609-seniors-1` (base `s6/start`, cabecera `RolandMartinez:auditoria-rm`) con el commit `8d967f0`. Después añadió este registro en un segundo commit.
