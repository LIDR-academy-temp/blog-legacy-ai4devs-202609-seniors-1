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

## Prompt 1 - Verificación del acceso al historial de Git

**Modelo:** sonnet (claude-sonnet-5-5)
**Herramienta:** Claude Code

```
Resume los últimos commits de este repositorio. Indica el hash corto y qué incluye cada commit. No modifiques ningún archivo.
```

**Qué salió:**

Claude Code consultó el historial del repositorio e identificó tres commits:

- `a87a414`: actualización de las referencias a los repositorios de la edición del curso en el README.
- `b5da02a`: ampliación de la documentación con requisitos técnicos, instrucciones de ejecución y detalles de la funcionalidad de comentarios.
- `e0e0943`: importación inicial del proyecto, incluyendo backend, frontend, migraciones, tests y configuración del agente.

También informó de un cambio local sin commitear en `prompts.md`.

**Observaciones:**

Se comprobó que el agente puede consultar el historial de Git y obtener información sobre la evolución del proyecto.

No se modificó ningún archivo durante esta consulta.

---

## Prompt 2 - Exploración de la funcionalidad de comentarios

**Modelo:** sonnet (claude-sonnet-5-5)
**Herramienta:** Claude Code

```
Busca en el repositorio los archivos donde aparece el término "comentario". Muéstrame sus rutas, y los artefactos relacionados e indica brevemente qué función tiene cada uno y donde se referencian. Limítate a investigar y no modifiques archivos.
```

**Qué salió:** Claude Code informó de 13 archivos que contienen referencias al término buscado y organizó los resultados por backend, frontend, documentación y herramientas.

**1. Backend y base de datos**

- `backend/start/routes.ts`: concentra la lógica de comentarios y sus endpoints.
  - Líneas 75-76: recuento de comentarios mediante la condición `estado <> 'pendiente'`.
  - Líneas 150-171: el detalle del post obtiene comentarios con `estado = 'aprobado'`.
  - Líneas 217-229: otra consulta obtiene comentarios para el recuento de relacionados.
  - Líneas 293-333: creación de comentarios mediante POST.
  - Líneas 337-361: borrado mediante DELETE, con comparación del email sin distinguir mayúsculas.
  - Líneas 364-392: edición mediante PATCH, con comparación exacta del email y comprobación del estado `pendiente`.
- Migración de comentarios: define la tabla con un campo `estado` de texto libre.
- Seeder: contiene datos de ejemplo con diferentes variantes del estado (`aprobado`, `Aprobado`, `ok`). Esto nos confirma una falta de consistencia en el modelo de datos.
- `backend/database/schema.ts`: contiene el esquema generado de comentarios.
- `backend/tests/functional/comentarios.spec.ts`: contiene un único test funcional que comprueba la creación de un comentario.

**2. Frontend**

- `Comentarios.jsx`: muestra comentarios y respuestas y contiene el formulario de creación.
- `DetallePost.jsx`: utiliza el componente de comentarios y calcula su propio recuento.
- `ListaPosts.jsx` y `PostsRelacionados.jsx`: muestran recuentos de comentarios.
- `index.css`: contiene estilos específicos de comentarios.

**3. Documentación, reglas de negocio y herramientas**

Este apartado resulta especialmente relevante porque el agente identificó reglas explícitas que pueden servir como referencia para comprobar los comportamientos del código.

- `CLAUDE.md`: contiene las reglas de negocio 2, 5 y 6 relacionadas con comentarios, además de referencias a la matriz de tests. Claude señaló las líneas 129, 144 y 147.
- `README.md`: documenta los endpoints (líneas 45-52), las reglas de negocio (61-64) y las instrucciones de auditoría (100-105).
- `prompts.md`: contiene el registro de prompts utilizados.
- `.claude/hooks/guard-tests.mjs`: contiene una referencia textual a "comentario" que no está relacionada con la funcionalidad del blog.

**4. Posibles inconsistencias identificadas por el agente**

Claude destacó las siguientes:

- **Estados inconsistentes:** los recuentos utilizan `estado <> 'pendiente'`, mientras que el detalle de los artículos utiliza `estado = 'aprobado'`. Con los estados presentes en los datos de ejemplo, ambos criterios podrían producir resultados diferentes.
- **Posible incumplimiento de la regla de negocio 6:** según Claude, las respuestas de POST y PATCH devuelven registros completos que incluyen `autor_email`. El agente lo relaciona con la regla 6 documentada en `CLAUDE.md`.
- **Posible exposición adicional de información:** el agente señala que el detalle del post utiliza `select *` sobre comentarios, lo que también podría exponer el correo electrónico. Claude indicó expresamente que no había comprobado este punto.
- **Reglas de autorización diferentes:** las operaciones de edición y borrado realizan comparaciones distintas del correo electrónico y aplican condiciones diferentes.
- **Arquitectura de comentarios:** el agente observó que la funcionalidad concentra su lógica en las rutas y no dispone de repositorio, servicio, validador o transformer propios.
- **Pruebas limitadas:** el único test funcional localizado comprueba el caso de creación de un comentario, pero no los demás comportamientos de la funcionalidad.

**Observaciones:**

La exploración permitió localizar los principales archivos involucrados en la funcionalidad de comentarios y algunas de sus dependencias.

El aspecto más relevante fue la identificación de **reglas de negocio documentadas en `CLAUDE.md` y `README.md`**, que pueden utilizarse como referencia durante la auditoría.

Especialmente interesante resulta la posible contradicción entre las respuestas de la API y la regla de negocio 6, que el agente relaciona con la exposición del correo electrónico del autor.

Sin embargo, ninguno de estos posibles problemas se considera todavía un hallazgo verificado. Será necesario contrastar las reglas documentadas con las implementaciones concretas, comprobar las líneas señaladas y determinar sus consecuencias.

La ausencia de capas como servicios o repositorios tampoco demuestra, por sí sola, que exista un defecto.

El propio agente dejó expresamente sin verificar la posible exposición del email mediante el `select *` del detalle del post.

Y algo interesante que me ha dado una pista, tras el resultado obtenido en la respuesta a este prompt, ha sido que tras realizar un checkeo al archivo `backend/start/routes.ts`, he podido observar que las rutas concentran demasiada lógica de consultas SQL, reglas de autorización y construcción de respuestas, contraviniendo la separación de responsabilidades establecida en `CLAUDE.md`.

No se modificó ningún archivo durante esta consulta.

---

## Prompt 3 - Auditoría priorizada de comentarios

**Modelo:** sonnet (claude-sonnet-5-5)
**Herramienta:** Claude Code

```
Realiza una **auditoría priorizada de la funcionalidad de comentarios** de este proyecto, siguiendo las instrucciones del ejercicio en `README.md` y las reglas de ingeniería de `CLAUDE.md`.

### 1. Alcance

Audita exclusivamente la funcionalidad de comentarios de extremo a extremo: creación, edición, borrado y listado, incluyendo rutas de la API, base de datos y frontend.

Consulta las reglas de negocio, la arquitectura objetivo y los contratos existentes. Busca problemas funcionales, inconsistencias de datos, seguridad, validación, autorización y separación de responsabilidades (SRP).

No extiendas la auditoría al resto del sistema.

### 2. Identificación de hallazgos

- Distingue **hechos comprobables, riesgos derivados y opiniones**.
- Agrupa los defectos repetidos que tengan una causa común en un único hallazgo transversal.
- Exige para cada afirmación **archivo, líneas exactas y evidencia en el código**. Sin evidencia, no incluyas el hallazgo.
- Contrasta los problemas con las reglas de negocio documentadas.
- Diferencia lo demostrado mediante lectura del código de lo que necesita comprobación adicional.
- Deja las propuestas sin medir fuera de la tabla, indicando qué habría que comprobar para evaluarlas.

Prioriza una lista corta, concreta y verificable, no una enumeración exhaustiva de problemas.

### 3. Priorización

Analiza cada hallazgo mediante los cuatro ejes aprendidos:

1. **Daño:** qué se rompe y quién lo nota.
2. **Radio:** cuántos lugares concretos habría que modificar, identificándolos y contándolos.
3. **Reversibilidad:** si el arreglo puede deshacerse con un revert y por qué.
4. **Precedencia:** qué otros hallazgos bloquea, desbloquea o facilita.

Ordena los hallazgos siguiendo este criterio:

- **Primero, precedencia:** los arreglos que desbloquean o abaratan otros van antes.
- **Después, daño dividido por radio:** dentro de cada nivel de precedencia.

Si un problema grave necesita atención inmediata, pero su solución definitiva depende de otros cambios, divídelo en dos filas: **contención y arreglo definitivo**.

No asignes prioridades arbitrarias ni inventes dependencias.

### 4. Entregable obligatorio

Crea `docs/auditoria/comentarios.md` con una tabla Markdown de **exactamente seis columnas**:

| Qué es | Dónde está, con la línea que lo demuestra | Qué se rompe y quién lo nota | Cuántos sitios hay que tocar | ¿Se deshace con un revert? | ¿Bloquea o desbloquea a otro hallazgo? |

Requisitos:

- Una fila por hallazgo, ordenadas según el criterio anterior.
- Identificador único para cada hallazgo (C-01, C-02...).
- Evidencias verificables y radio contado, no estimado.
- Dependencias identificadas explícitamente.
- **Casillas en blanco cuando falte información**, sin completar mediante suposiciones.

Debajo de la tabla, deja preparado el apartado **Parte B — Las tres líneas**, con espacio para que yo registre:

1. Número de hallazgos devueltos por el agente y número comprobado personalmente.
2. Hallazgo cuya prioridad no supe decidir y motivo.
3. Afirmación del agente que no pude verificar.

No inventes mis comprobaciones ni mis reflexiones.

### 5. Restricciones

Realiza únicamente la auditoría. No modifiques código, tests, migraciones ni datos; no implementes soluciones ni hagas commits.

Solo puedes crear o modificar `docs/auditoria/comentarios.md`.

Al terminar, resume en el chat **cuántos hallazgos has identificado, cómo los has priorizado y qué queda pendiente de verificar personalmente**, para que podamos revisarlos uno a uno.
```

**Qué salió:**
- Claude Code generó `docs/auditoria/comentarios.md` con una auditoría priorizada de la funcionalidad de comentarios.
- Identificó 12 hallazgos (C-01 a C-12), documentados en una tabla con las seis columnas exigidas.
- Analizó problemas de cobertura de tests, arquitectura, seguridad, validación, autorización, integridad de datos y comportamiento del frontend.
- Propuso una priorización basada primero en las dependencias entre hallazgos y después en el daño frente al radio de los cambios.
- Dividió la exposición de correos electrónicos en dos actuaciones: C-03 (contención inmediata) y C-06 (arreglo definitivo mediante Transformer).
- Dejó algunas casillas incompletas cuando no disponía de información suficiente.
- Separó las propuestas sin medir de los hallazgos y preparó la Parte B para completarla personalmente.
- El análisis se realizó mediante inspección del código y consultas de solo lectura a SQLite, sin ejecutar la API, la web ni los tests. No se modificó código funcional.

**Observaciones:**
- Aunque la auditoría presenta el formato solicitado, hay varios aspectos que quiero revisar antes de aceptar la priorización propuesta:
- Precedencia de C-01 y C-02: Claude sitúa la falta de tests y los problemas arquitectónicos por delante de los demás hallazgos. Quiero comprobar si representan dependencias técnicas reales o principalmente requisitos del proceso de refactorización.
- Contención de C-03: se propone eliminar inmediatamente la exposición del correo electrónico, pero aparece después de C-01 y C-02. Conviene valorar si esa contención puede aplicarse independientemente para reducir el daño cuanto antes.
- Cantidad de hallazgos: las 12 filas resultan numerosas para el tiempo disponible. Priorizaré la verificación de una lista reducida, sin intentar completar todas las comprobaciones.
- Evidencias pendientes: varias consecuencias se han deducido mediante lectura del código y todavía necesitan comprobarse en ejecución. No deben considerarse verificadas personalmente.
- No seguí literalmente la lista de comprobaciones propuesta por Claude Code. Decidí revisar los hallazgos individualmente y seleccionar los más relevantes según el tiempo disponible.
- Comprobé personalmente C-01, C-02, C-03, C-04, C-05 y C-07, contrastando las afirmaciones con los archivos del proyecto y, en C-03 y C-07, también con respuestas reales de la API.
- Dejé C-06 sin verificar por completo porque representa el arreglo definitivo de la exposición de datos detectada en C-03.
- Preferí comprobar primero que la fuga existía y analizar su contención inmediata, sin investigar todavía todos los cambios necesarios para implementar un Transformer.
- La experiencia me permitió distinguir entre problemas demostrados, riesgos pendientes de ejecución y dependencias técnicas reales frente a requisitos del proceso de refactorización.
- Estas observaciones comenzaron como dudas sobre la propuesta del agente. Durante la verificación confirmé varios hallazgos y cuestioné algunas dependencias de la clasificación inicial, aunque quedaron aspectos sin comprobar por la limitación de tiempo.