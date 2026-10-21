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

**Modelo:** Claude Opus 5.5 (`claude-opus-5-5`)
**Herramienta:** Claude Code

```
Estamos trabajando en el ejercicio de auditoría del proyecto Blog Legacy.

Antes de comenzar, necesito que verifiques el estado actual del repositorio.

Indica la rama actual.

Resume los últimos commits relevantes.

Identifica si hay cambios locales pendientes y en qué archivos.

Confirma si puedes ejecutar comandos de Git y buscar archivos dentro del proyecto.

Indica si existe el archivo docs/auditoria/comentarios.md y si prompts.md contiene una plantilla.

No modifiques archivos, no cambies de rama, no hagas commits y no corrijas ningún problema. No asumas que el repositorio está limpio: compruébalo mediante comandos y muestra la evidencia disponible.

Si algo no se puede verificar, indícalo explícitamente.

realiza login
run /login
```

**Qué salió:** funcionó a la primera: `/login` correcto, rama `s6/start` limpia, `comentarios.md` no existía y `prompts.md` tenía la plantilla.

---

## Prompt 2

**Modelo:** Claude Opus 5.5 (`claude-opus-5-5`)
**Herramienta:** Claude Code

```
crea el archivo comentario.md donde se indica.
```

**Qué salió:** el nombre del prompt no coincidía con el del README; el agente creó `docs/auditoria/comentarios.md` (plural) y avisó del cambio. Solo la plantilla vacía.

---

## Prompt 3

**Modelo:** Claude Opus 5.5 (`claude-opus-5-5`)
**Herramienta:** Claude Code

```
Audita exclusivamente la funcionalidad de comentarios del proyecto Blog Legacy.

Necesito identificar cómo funciona de punta a punta en estas tres capas:

API: rutas y operaciones para crear, editar, borrar y listar comentarios.

Base de datos: tablas, campos, relaciones y restricciones relacionadas con comentarios.

Frontend: componentes y llamadas a la API que permiten mostrar y gestionar comentarios.

Busca directamente en el repositorio. Para cada elemento encontrado, proporciona la ruta del archivo y las líneas relevantes.

No audites posts, autores, categorías ni etiquetas, salvo cuando una relación directa sea imprescindible para explicar un hallazgo de comentarios.

No modifiques archivos. No inventes rutas, comportamientos, relaciones ni líneas de código. Si no encuentras alguna operación, indica que no has podido localizarla y explica dónde buscaste.

Devuelve primero el mapa técnico, sin priorizar ni proponer soluciones todavía.
```

**Qué salió:** mapa técnico de API, base de datos, frontend y tests con líneas; indicó lo no localizado (endpoint de moderación y de listado propio).

---

## Prompt 4

**Modelo:** Claude Opus 5.5 (`claude-opus-5-5`)
**Herramienta:** Claude Code

```
sí, prioriza los hallazgos en la tabla
```

**Qué salió:** primera tabla con 14 hallazgos (F1–F14) escrita en el archivo, sin verificación adversarial todavía.

---

## Prompt 5

**Modelo:** Claude Opus 5.5 (`claude-opus-5-5`)
**Herramienta:** Claude Code

```
Realiza una auditoría de código de la funcionalidad de comentarios de Blog Legacy, considerando únicamente su API, su persistencia en base de datos y el frontend relacionado.

Busca posibles defectos de comportamiento, integridad de datos, autorización, validación, consistencia entre capas y manejo de errores, siempre que exista evidencia en el código.

Para cada posible hallazgo, entrega:

Descripción concreta del problema.

Archivo y líneas exactas que permitan localizar la evidencia.

Explicación del escenario que provocaría el problema.

Efecto observable y quién resultaría afectado.

Número de archivos o sitios de código que habría que modificar, únicamente si puedes contarlos.

Si un revert sería suficiente para deshacer el cambio y por qué.

Si el hallazgo bloquea o desbloquea otro hallazgo, indicando cuál, si existe evidencia para establecerlo.

Distingue los defectos demostrados de las sospechas y de las propuestas de mejora que todavía necesitan medición.

No afirmes que un comportamiento ocurre en producción si solo has inspeccionado el código. No inventes pruebas ejecutadas, resultados, líneas ni relaciones de dependencia.

No modifiques ningún archivo ni implementes soluciones. Devuelve candidatos a hallazgos, no una auditoría definitiva. No incluyas funcionalidades ajenas a comentarios.
```

**Qué salió:** 20 candidatos (12 defectos, 2 sospechas, 6 mejoras por medir) en el chat.

---

## Prompt 6

**Modelo:** Claude Opus 5.5 (`claude-opus-5-5`)
**Herramienta:** Claude Code

```
Ahora actúa como revisor adversarial de los candidatos de la auditoría de comentarios que acabas de generar.

No des por correcto ningún hallazgo por el mero hecho de que aparezca en tu respuesta anterior.

Para cada candidato:

Abre el archivo real y comprueba el fragmento citado.

Confirma las líneas actuales y si el fragmento respalda exactamente la afirmación.

Sigue el flujo de ejecución relacionado cuando sea necesario, sin salir de la funcionalidad de comentarios.

Determina si el escenario descrito se desprende del código o si requiere una prueba adicional.

Indica qué comprobaste directamente y qué no pudiste comprobar.

Señala los hallazgos duplicados o las afirmaciones que mezclan problemas independientes.

Clasifica cada candidato como:

Verificado mediante inspección del código.

Refutado por la evidencia.

No verificable con la evidencia disponible.

Requiere una prueba adicional.

No inventes resultados de ejecución, pruebas, líneas ni comportamientos. No modifiques archivos. No completes datos faltantes mediante suposiciones.

Devuelve una tabla de verificación con la evidencia y las limitaciones de cada candidato.
```

**Qué salió:** refutó un candidato propio (cuerpo de solo espacios), corrigió alcances y citas, y marcó duplicados y mezclas.

---

## Prompt 7

**Modelo:** Claude Opus 5.5 (`claude-opus-5-5`)
**Herramienta:** Claude Code

```
Muestrame los archivos para validar los hallazgos identficados, comencemos con el primero.
```

**Qué salió:** evidencia del hallazgo 1 (correo en la respuesta pública) con fragmentos y un `curl` para comprobarlo.

---

## Prompt 8

**Modelo:** Claude Opus 5.5 (`claude-opus-5-5`)
**Herramienta:** Claude Code

```
sigue con el hallazgo
```

**Qué salió:** evidencia del hallazgo 3 (clave de moderación en el código e historial de git).

---

## Prompt 9

**Modelo:** Claude Opus 5.5 (`claude-opus-5-5`)
**Herramienta:** Claude Code

```
on base exclusivamente en los candidatos y en la evidencia de verificación disponible, ayúdame a priorizar los hallazgos de la auditoría de comentarios.

Aplica estrictamente estos criterios:

Primero, la precedencia: coloca antes lo que reduce el coste o desbloquea otro hallazgo.

Dentro de cada nivel de precedencia, compara el daño con el radio afectado.

Si un problema tiene un daño alto, pero su arreglo de fondo debe esperar, sepáralo en dos hallazgos cuando proceda: contención inmediata y arreglo definitivo.

Las propuestas que todavía no se han medido deben quedar fuera de la tabla principal, indicando qué habría que medir.

No incluyas como hechos verificados los candidatos refutados o no comprobados.

No inventes dependencias entre hallazgos. Si no hay evidencia de que uno bloquee o desbloquee a otro, indícalo.

Para cada hallazgo conserva estos seis campos:

Qué es.

Dónde está, con archivo y líneas que lo demuestran.

Qué se rompe y quién lo nota.

Cuántos sitios hay que tocar, contados y no estimados.

Si se deshace con un revert, sí o no, y por qué.

Si bloquea o desbloquea a otro hallazgo, indicando cuál.

No modifiques el código. Devuelve una propuesta de orden y explica brevemente las decisiones difíciles, especialmente cuando la gravedad y la precedencia entren en conflicto. Si la evidencia no permite decidir, deja explícita la incertidumbre.
```

**Qué salió:** propuesta de orden con 18 filas, contención y arreglo definitivo separados, 4 mejoras fuera de tabla y 6 decisiones difíciles.

---

## Prompt 10

**Modelo:** Claude Opus 5.5 (`claude-opus-5-5`)
**Herramienta:** Claude Code

```
Toma ok como aprobado,
```

**Qué salió:** cambiaron 4 filas; el recuento del listado (#9) bajó de gravedad.

---

## Prompt 11

**Modelo:** Claude Opus 5.5 (`claude-opus-5-5`)
**Herramienta:** Claude Code

```
Si por favor
```

**Qué salió:** volcó la priorización final en `docs/auditoria/comentarios.md`.

---

## Prompt 12

**Modelo:** Claude Opus 5.5 (`claude-opus-5-5`)
**Herramienta:** Claude Code

```
Redacta el entregable de la auditoría de comentarios en docs/auditoria/comentarios.md utilizando únicamente los hallazgos y las comprobaciones que realmente se realizaron durante esta sesión.

Requisitos:

La auditoría debe cubrir exclusivamente comentarios.

Incluye una tabla con estas seis columnas:

Qué es.

Dónde está, con líneas de evidencia.

Qué se rompe y quién lo nota.

Cuántos sitios hay que tocar.

¿Se deshace con un revert?

¿Bloquea o desbloquea a otro hallazgo?

Respeta la priorización por precedencia y, dentro de cada nivel, daño dividido por radio.

Separa la contención del arreglo definitivo cuando corresponda.

Coloca las propuestas no medidas fuera de la tabla e indica qué habría que medir.

Deja las casillas sin comprobar vacías. No rellenes de memoria los datos que falten.

Debajo de la tabla escribe estas tres respuestas, con información real de la sesión:

Número de hallazgos devueltos por el agente y número de hallazgos que comprobé personalmente abriendo los archivos.

El hallazgo cuya prioridad no supe decidir y una frase que explique qué dos criterios entraron en conflicto.

Una afirmación del agente que no pude verificar ni confirmar ni refutar.

Si todavía no tengo información real para alguna de las tres respuestas, deja un marcador explícito para que yo lo complete. No inventes respuestas.

No modifiques código fuente, pruebas ni otros archivos. Escribe únicamente docs/auditoria/comentarios.md y después muéstrame el contenido completo para revisarlo.
```

**Qué salió:** entregable reescrito; casilla de sitios del #10 vacía y tres marcadores pendientes en la Parte B.

---

## Prompt 13

**Modelo:** Claude Opus 5.5 (`claude-opus-5-5`)
**Herramienta:** Claude Code

```
Revisa las tres líneas obligatorias de docs/auditoria/comentarios.md.

Línea 1:

Registra que el agente devolvió 20 candidatos.

Indica que la tabla final contiene 18 hallazgos, 4 propuestas pendientes de medir y 3 candidatos excluidos, como consta en el archivo.

Registra que yo comprobé personalmente 4 hallazgos.

Línea 2:

Ayúdame a expresar el conflicto de priorización de forma concreta.

Utiliza como candidato el hallazgo #10, la ausencia de una operación para aprobar comentarios, únicamente si la evidencia disponible permite explicar el conflicto entre su impacto y su dependencia de otros hallazgos.

No afirmes que este fue definitivamente el hallazgo que me generó dudas si no existe evidencia suficiente.

Si falta confirmar mi criterio personal, conserva un marcador explícito para que yo lo complete.

Línea 3:

Identifica una afirmación concreta del agente que no se haya podido verificar ni confirmar ni refutar.

La posible condición de carrera en la creación de comentarios y la afirmación sobre zonas horarias son candidatos mencionados en la sesión.

No presentes como propia una comprobación que yo no realicé.

Si no es posible determinar cuál de ellas quedó sin verificar personalmente, deja un marcador explícito.

No inventes respuestas para completar los campos. No cambies el número de hallazgos ni los resultados de la auditoría sin evidencia.
```

**Qué salió:** Parte B completada; dos marcadores para confirmar mi criterio personal (líneas 2 y 3).

---

## Prompt 14

**Modelo:** Claude Opus 5.5 (`claude-opus-5-5`)
**Herramienta:** Claude Code

```
Revisa la plantilla existente y complétala respetando su estructura.

Incluye los prompts que realmente se ejecutaron durante el ejercicio, con:

Texto literal del prompt.

Herramienta utilizada.

Modelo utilizado, solo si puede confirmarse.

Resultado breve de cada ejecución, si la plantilla lo contempla.

Incluye los intentos que no funcionaron, cuando corresponda.

No reconstruyas como literales los prompts que no puedas recuperar. No atribuyas a la sesión prompts que solo fueron propuestos posteriormente como ejemplos. Si el texto exacto o el modelo no están disponibles, marca el dato como pendiente en lugar de inventarlo.

Respeta las exclusiones que ya se hayan indicado durante la sesión y no reincorpores prompts expresamente excluidos.
```

**Qué salió:** este archivo.
