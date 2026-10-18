## Prompt 0

**Modelo:** Opus 5.5 xHigh
**Herramienta:** Claude Code

```
Revisa los últimos 5 commits del repositorio y resume brevemente qué cambios introdujeron.

Después, busca los archivos relacionados con la funcionalidad de comentarios e identifica cuáles pertenecen al backend, frontend y base de datos.

No modifiques ningún archivo. No ejecutes cambios sobre la base de datos. Solo quiero comprobar que puedes explorar el repositorio y ejecutar comandos de Git.
```

**Qué salió:** Identificó archivos de backend, frontend y base de datos; no modificó el repositorio.

---

## Prompt 1

**Modelo:** Opus 5.5 xHigh
**Herramienta:** Claude Code

```
Actúa como auditor técnico de un proyecto legacy.

Analiza exclusivamente la funcionalidad de comentarios del blog, de extremo a extremo: backend/API, base de datos y frontend. Incluye crear, editar, eliminar y listar comentarios.

Objetivo: identificar posibles defectos, inconsistencias, riesgos de seguridad, problemas de integridad de datos y ausencia de pruebas relevantes.

Para cada posible hallazgo, proporciona:
1. Identificador único.
2. Descripción concreta del problema.
3. Archivo y líneas exactas que sustentan la sospecha.
4. Comportamiento actual y comportamiento esperado, indicando la fuente de esa expectativa.
5. Consecuencia potencial y usuario o consumidor afectado.
6. Estado: confirmado por evidencia estática o pendiente de verificación.

Consulta CLAUDE.md y las instrucciones del proyecto, pero no asumas que el código cumple sus reglas.

No inventes comportamientos ni referencias. No repitas el mismo problema en múltiples hallazgos. Distingue hechos comprobables de hipótesis.

No priorices todavía, no modifiques archivos, no ejecutes comandos que alteren datos y no corrijas ningún problema.

Al final, indica el número total de hallazgos identificados.
```

**Qué salió:** Claude identificó 27 posibles hallazgos y clasificó 22 como confirmados por evidencia estática. Posteriormente comprobé personalmente seis mediante lectura del código.

---

## Prompt 2

**Modelo:** Opus 5.5 xHigh
**Herramienta:** Claude Code

```
Vamos a verificar una selección de los hallazgos anteriores.

Trabaja exclusivamente con:
COM-01, COM-04, COM-05, COM-09, COM-13 y COM-26.

Para cada uno:
1. Muestra la ruta y el intervalo exacto de líneas.
2. Incluye un fragmento mínimo del código relevante.
3. Explica qué demuestra ese código y qué NO demuestra.
4. Identifica las otras rutas o componentes que habría que inspeccionar para confirmar el impacto.
5. Señala qué afirmaciones requieren una prueba de ejecución o información externa.

No marques nada como verificado por mí.
No priorices los hallazgos.
No modifiques archivos ni datos.
No ejecutes operaciones destructivas.

Quiero utilizar tu respuesta como guía para abrir yo mismo los archivos y comprobar las afirmaciones.
```

**Qué salió:** Proporcionó referencias de código que utilicé para verificar personalmente seis hallazgos.

---

## Prompt 3

**Modelo:** Opus 5.5 xHigh
**Herramienta:** Claude Code

```
Estamos preparando la auditoría priorizada de comentarios.

He revisado personalmente las evidencias estáticas de COM-01, COM-04, COM-05, COM-09, COM-13 y COM-26.

Para estos seis hallazgos exclusivamente:

1. Identifica las dependencias entre hallazgos.
2. Cuenta los archivos y lugares concretos que habría que modificar para una corrección mínima. Enuméralos y distingue cambios obligatorios de opcionales.
3. Determina si un git revert sería suficiente para deshacer la corrección, considerando también cambios en datos o migraciones.
4. Identifica quién sufriría el daño y cuál sería la consecuencia concreta.
5. Propón un orden por precedencia y, dentro de cada nivel, por daño dividido por radio de cambio.
6. Cuando un riesgo grave requiera una corrección de fondo compleja, separa contención inmediata y corrección definitiva.

No presentes estimaciones como hechos comprobados.
No agregues hallazgos nuevos.
No modifiques código.
No escribas todavía el documento final.
Señala explícitamente las casillas que no puedas justificar con evidencia.
```

**Qué salió:** Propuso dependencias, radios de cambio y reversibilidad, distinguiendo hechos, estimaciones y falta de evidencia.



