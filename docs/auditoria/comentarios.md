# Auditoría priorizada — Comentarios

**Alcance:** crear, listar, editar y eliminar comentarios (API, SQLite y frontend).
**Método:** auditoría estática asistida por Claude Code, seguida de comprobación manual de seis hallazgos. Sin cambios de código ni pruebas de explotación.

**Criterio:** primero precedencia; dentro del mismo nivel, daño potencial dividido por radio de cambio. Los radios son estimaciones de corrección mínima, no cambios implementados.

| Prioridad / hallazgo | Qué ocurre | Evidencia | Impacto / afectado | Radio mínimo | ¿Revert? | Dependencia |
|---|---|---|---|---|---|---|
| 1. COM-26 | Pruebas insuficientes para cambiar comportamiento con seguridad | `backend/tests/functional/comentarios.spec.ts:15–43` | Regresiones no detectadas; mantenedores y usuarios | 1 archivo de tests (estimado) | Sí | Habilita las demás correcciones |
| 2. COM-01 | GET del post devuelve comentarios con todos sus campos, incluido el correo | `backend/start/routes.ts:152–155,166–174` | Exposición de correo; comentaristas | 2 archivos / 2 lugares (estimado) | Sí para código; no revierte filtraciones previas | Agrava COM-04 |
| 3. COM-05 | Secreto de moderación literal en el repositorio | `backend/start/routes.ts:25,349–351` | Posibles borrados indebidos; comentaristas | 4–5 archivos + configuración externa (estimado) | Parcial: rotación e historial no se revierten | Precede a contención de COM-04 |
| 4. COM-04 — contención | DELETE acepta el correo proporcionado como prueba de autoría | `backend/start/routes.ts:339–359` | Borrado de comentarios ajenos; autores | 2 archivos incluyendo tests (estimado) | Sí | Depende de proteger COM-05; reduce riesgo de COM-13 |
| 5. COM-13 — contención | Borrar un padre deja respuestas sin representación visible | `backend/start/routes.ts:359`; `frontend/src/componentes/Comentarios.jsx:50–64`; migración `:20` | Respuestas invisibles; autores y lectores | 2 archivos incluyendo tests (estimado) | Sí, si solo se bloquea el borrado con respuestas | Se reduce su exposición al contener COM-04 |
| 6. COM-09 | Listados cuentan estados distintos de `pendiente`, detalle solo `aprobado` | `backend/start/routes.ts:75–76,153,222–224` | Recuentos contradictorios; lectores | 3 lugares incluyendo tests (estimado) | Sí, si solo se modifica código | Solución de datos pendiente de COM-10 |

**Correcciones definitivas aplazadas:** COM-04 requiere decidir un mecanismo de autenticación o prueba de propiedad; COM-13 requiere definir la política de borrado de respuestas; COM-09 requiere aclarar el significado de los estados `ok` y `Aprobado`. Las migraciones o modificaciones de datos no se consideran reversibles mediante un simple `git revert`.

**Sin verificar:** explotabilidad HTTP de COM-04, alcance real de exposición de COM-01, despliegues que utilizan COM-05, existencia de respuestas huérfanas y significado funcional de estados alternativos.

## Reflexiones obligatorias

1. **Hallazgos:** Claude devolvió 27; comprobé personalmente 6 mediante lectura de las líneas de código. No ejecuté pruebas específicas para confirmar su explotación.
2. **Conflicto de prioridad:** COM-04 presenta riesgo de seguridad, mientras COM-26 habilita una corrección más segura. Elegí caracterizar primero los flujos afectados, sin retrasar innecesariamente la contención.
3. **Afirmación sin confirmar ni refutar:** No comprobé si COM-03 puede devolver el comentario equivocado cuando dos solicitudes POST se ejecutan concurrentemente.
