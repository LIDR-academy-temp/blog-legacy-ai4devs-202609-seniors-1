# Promps


## Prompt 1

**Modelo:** Opus 5.5 (Amazon Bedrock)
**Herramienta:** Claude Code

```
Analiza este repositorio sin modificar ningún archivo. Resume los últimos 5 commits de Git e identifica los archivos relacionados con la funcionalidad de comentarios en backend, frontend y base de datos. Indica las rutas exactas. No propongas correcciones todavía.
```

**Qué salió:** El repositorio solo tiene 3 commits, no 5, y así se indicó. La búsqueda inicial de archivos no llegó a ejecutarse por un error de shell (`(eval):1: == not found`) y hubo que relanzarla. Devolvió las rutas de comentarios en base de datos, backend y frontend. No modificó nada.

## Prompt 2

**Modelo:** Opus 5.5 (Amazon Bedrock)
**Herramienta:** Claude Code

```
Audita exclusivamente la funcionalidad de comentarios de este blog, incluyendo crear, editar, eliminar y listar comentarios en backend, frontend y base de datos.

No modifiques ningún archivo ni corrijas problemas.

Identifica posibles defectos, inconsistencias, riesgos de seguridad y problemas de integridad de datos.

Para cada hallazgo proporciona:
1. Descripción concreta del problema.
2. Archivo y números de línea que lo demuestran.
3. Qué comportamiento se rompe y quién resulta afectado.
4. Número exacto de archivos o sitios que habría que modificar.
5. Si puede revertirse con git revert y por qué.
6. Si bloquea o desbloquea otro hallazgo identificado.

Distingue claramente entre hallazgos comprobados, hipótesis pendientes y propuestas que requieren medición. No inventes evidencias.

Considera como línea base que make test ejecutó una prueba funcional de comentarios y falló con SqliteError: no such table: posts. No corrijas ese error.

Devuelve una lista corta de hallazgos candidatos. Todavía no escribas el documento final.
```

**Qué salió:** Devolvió 12 hallazgos candidatos (H0-H11), cada uno con los seis campos y una etiqueta de comprobado, hipótesis o medición pendiente. Todo salió de leer código y la semilla. No se ejecutaron la API ni los tests, porque `backend/tmp/db.sqlite3` ocupa 0 bytes y sembrarla habría cambiado datos. Las cifras de recuento y la carrera del POST (H9) quedaron sin medir. Algunos «números exactos de sitios» son mínimos estimados, no exactos.

## Prompt 3

**Modelo:** Opus 5.5 (Amazon Bedrock)
**Herramienta:** Claude Code

```
Crea docs/auditoria/comentarios.md con una auditoría parcial de la funcionalidad de comentarios. El agente devolvió 12 hallazgos (H0-H11), pero yo únicamente comprobé abriendo el código H1 (routes.ts:153,171,329-333), H2 (routes.ts:25,349-351) y H5 (routes.ts:75-76,153,223). No presentes los otros como verificados por mí. Incluye una matriz breve con las seis columnas exigidas: qué es, dónde está con línea, qué se rompe y quién lo nota, sitios exactos a tocar, si se deshace con git revert y dependencias. Deja en blanco cualquier dato no comprobado. Prioriza primero por precedencia y luego por daño dividido entre radio de impacto; separa contención y arreglo de fondo cuando corresponda. Fuera de la tabla coloca hipótesis no medidas. Debajo escribe exactamente tres líneas: (1) 12 hallazgos devueltos y 3 comprobados por mí; (2) no pude decidir entre contener H2 por su riesgo y atender H0 porque desbloquea las pruebas; (3) no pude verificar si H9 devuelve comentarios de otra persona bajo concurrencia. No modifiques el código de la aplicación.
```

**Qué salió:** Creó `docs/auditoria/comentarios.md`. La matriz tiene filas completas para H1, H2 y H5, con contención y arreglo de fondo separados en H1 y H2. Las filas de H0, H3, H4, H6, H7 y H8 llevan la marca «sin comprobar» y las celdas en blanco. H2 (contención) y H0 quedaron empatados en el primer puesto, sin resolver. Las hipótesis H9-H11 van fuera de la tabla y el documento cierra con las tres líneas pedidas. Limitaciones: la columna de git revert usa datos que comprobó el agente, no el usuario (`git log`, `.gitignore`), y el orden por daño ÷ radio es cualitativo, no medido.
