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

**Modelo:** Sonnet High
**Herramienta:** Claude Code

```
# Prompt agéntico — Auditoría priorizada de la feature de comentarios

## 1. CONTEXTO / ROLE

Actúa como **auditor senior de software**, especializado en análisis de código, seguridad, integridad de datos y evaluación de riesgos en aplicaciones web.

La auditoría debe adoptar un enfoque adversarial, basado exclusivamente en evidencia verificable del repositorio y orientado a identificar y priorizar defectos reales.

## 2. OBJETIVO / TAREA

Realiza una **auditoría técnica integral, pero estrictamente limitada a la feature de comentarios** del blog, considerando su comportamiento de extremo a extremo en tres capas:

- **API:** operaciones de creación, edición, eliminación y listado de comentarios.
- **Base de datos:** tabla de comentarios, relaciones, restricciones y mecanismos de persistencia.
- **Web:** interfaces y comportamientos mediante los cuales los usuarios interactúan con los comentarios.

Identifica los hallazgos, evalúa su impacto y dependencias, y construye una **auditoría priorizada y accionable**.

Registra el resultado exclusivamente en:

`docs/auditoria/comentarios.md`

## 3. CRITERIOS DE ÉXITO EXPLÍCITOS

### 3.1. Cobertura y evidencia

- Se han examinado las tres capas de la feature y sus interacciones.
- Cada hallazgo corresponde a un defecto o riesgo concreto, sustentado en el código existente.
- Cada hallazgo identifica el archivo y las líneas específicas que demuestran el problema.
- Se distingue entre defectos comprobados, riesgos sustentados y aspectos que requieren validación adicional.

### 3.2. Priorización

Los hallazgos se ordenan mediante dos criterios, aplicados en esta secuencia:

1. **Precedencia:** primero los defectos cuya corrección desbloquea, simplifica o condiciona la resolución de otros hallazgos.
2. **Daño dividido por el radio:** dentro de cada nivel de precedencia, priorizar según el impacto concreto del defecto en relación con el alcance de la intervención necesaria para corregirlo.

Cuando un hallazgo presenta **alto daño y baja precedencia**, se divide en dos hallazgos independientes cuando corresponda:

- **Contención inmediata:** medida que interrumpe o limita el daño actual.
- **Corrección definitiva:** solución de fondo, respetando sus dependencias y precedencia.

La priorización debe ser explícita y justificable a partir de la evidencia recopilada.

### 3.3. Formato obligatorio de los hallazgos

Presenta una tabla ordenada por prioridad, con **una fila por hallazgo y exactamente seis columnas**:

| Columna | Contenido obligatorio |
|---|---|
| Qué es | Descripción del hallazgo en una sola frase. |
| Dónde está | Archivo y líneas específicas que sustentan el hallazgo. |
| Qué se rompe y quién lo nota | Consecuencia concreta y actor afectado; indicar «Nadie todavía» cuando corresponda. |
| Cuántos sitios hay que tocar | Número de ubicaciones concretas que requieren modificación, contadas a partir del repositorio. |
| ¿Se deshace con un revert? | Sí o no; cuando sea no, justificar por qué la reversión no elimina el efecto. |
| ¿Bloquea o desbloquea a otro hallazgo? | Identificar los hallazgos relacionados mediante sus identificadores; usar «A ninguno» cuando no existan dependencias. |

Asigna identificadores únicos a los hallazgos para facilitar las referencias cruzadas, sin añadir columnas.

**Ningún hallazgo puede incorporarse a la tabla si carece de evidencia de código o de un conteo verificable de los sitios que deben modificarse.**

### 3.4. Propuestas sin medición suficiente

Las propuestas cuya efectividad, alcance o impacto todavía no puedan determinarse con evidencia suficiente deben documentarse **fuera de la tabla**, en una sección independiente denominada «Pendiente de medición».

Para cada una, indicar qué debe medirse o verificarse antes de considerar su incorporación a la auditoría priorizada.

### 3.5. Entregable

El archivo `docs/auditoria/comentarios.md` debe:

- Ser autónomo, comprensible y utilizable para planificar las correcciones.
- Presentar los hallazgos en orden de ejecución recomendado.
- Permitir rastrear cada hallazgo hasta su evidencia.
- Distinguir las medidas de contención de las correcciones definitivas.
- Conservar únicamente información pertinente para tomar decisiones sobre la feature de comentarios.

## 4. RESTRICCIONES / ANTIPATTERNS

- **No ampliar el alcance** a las otras features del blog: posts, autores, categorías o etiquetas, salvo los puntos de interacción indispensables para evaluar comentarios.
- No convertir la auditoría en una evaluación general de arquitectura, calidad o deuda técnica del sistema.
- No modificar código, esquemas, configuraciones ni comportamiento de la aplicación.
- No implementar correcciones: el trabajo consiste exclusivamente en auditar y documentar.
- No presentar hallazgos especulativos como defectos comprobados.
- No inventar rutas, números de línea, cantidades de modificaciones, impactos ni dependencias.
- No utilizar estimaciones en la columna «Cuántos sitios hay que tocar».
- No agrupar múltiples defectos independientes en un solo hallazgo.
- No anteponer criterios subjetivos de gravedad al orden de precedencia establecido.
- No recomendar refactorizaciones generales ni mejoras que excedan el problema identificado.
- No incorporar propuestas sin evidencia suficiente en la tabla principal.
- No sustituir el archivo solicitado por una respuesta extensa en el chat.

## 5. RECURSOS

Utiliza como fuentes primarias:

- El código fuente del repositorio.
- Las rutas, controladores, servicios y validaciones de la API relacionadas con comentarios.
- Los modelos, migraciones, restricciones y relaciones de persistencia.
- Los componentes, formularios y mecanismos de consumo de la API en la web.
- Las pruebas automatizadas y documentación técnica existentes, cuando resulten pertinentes.

**Referencia metodológica:** criterio de priorización del mentor: precedencia primero y daño dividido por el radio dentro de cada nivel.

**Destino del entregable:** `docs/auditoria/comentarios.md`.

## 6. CLARIFICACIÓN

Trabaja de manera autónoma utilizando las convenciones y la estructura existente del repositorio.

Solicita aclaración únicamente si una ambigüedad impide determinar de forma verificable el alcance o la evidencia de un hallazgo relevante.

En los demás casos, documenta explícitamente las incertidumbres sin inventar información y continúa con la auditoría.

Al finalizar, informa brevemente en el chat la ruta del archivo generado, la cantidad de hallazgos documentados y cualquier limitación significativa encontrada. **La auditoría completa debe permanecer en el archivo, no en el chat.**
```

**Qué salió:** (opcional, una línea) funcionó a la primera / tuve que insistir / me inventó una ruta que no existe.
```
La auditoría está escrita en docs/auditoria/comentarios.md. Tiene 20 hallazgos, cada uno con sus seis columnas, y es lo único que cambió en el repositorio.

Verifiqué los hallazgos ejecutando la API sobre una copia aislada del backend, no sobre la tuya.

- Hallazgos: 18 están comprobados y se ven en la ejecución o en el código. C-18, el contador de la web, lo comprobé solo leyendo el código. C-19 es un riesgo sustentado: no se reprodujo en 40 envíos concurrentes.
- Prioridad: el orden sigue precedencia primero y daño ÷ radio dentro de cada nivel. El archivo incluye el grafo de dependencias. La contención y la corrección definitiva van separadas en tres problemas: el correo expuesto (C-02 → C-16), la clave de moderación (C-03 → C-15) y los estados sin normalizar (C-04 → C-08).
- Lo más serio:
  - El detalle del post devuelve el correo de cada comentarista (C-02).
  - Cualquiera puede borrar un comentario ajeno usando ese correo público (C-07).
  - La clave de moderación está escrita en el código (C-03).
  - Tres comentarios aprobados no se ven en el detalle del post, pero sí cuentan en el listado (C-04).
- Pendiente de medición: 10 propuestas quedan fuera de la tabla, cada una con lo que hay que medir. La más importante es P-01: no existe ninguna ruta que apruebe comentarios. Hay que confirmar si existe un panel externo; si no, todo comentario nuevo quedaría invisible para siempre.

Limitaciones:
- La web no se ejecutó en un navegador.
- No hay datos de producción.
- La base local backend/tmp/db.sqlite3 no está versionada, así que los conteos de filas describen esa copia.
- Detuve los dos servidores temporales que levanté (puertos 3498 y 3499). Un pkill amplio fue denegado por el sistema, así que cerré cada proceso por su PID.

El README pide además una «Parte B» de tres líneas, tuyas: cuántos hallazgos devolvió el agente y cuántos comprobaste, el hallazgo del que dudas la prioridad, y algo que no pudiste verificar. No las escribí porque no estaban en el prompt.
```