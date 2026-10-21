# Parte A: la auditoría


| Qué es. | Dónde está | Qué rompe, y quien lo nota | Cuántos sitios hay que tocar. | ¿Se deshace con un revert? | ¿Bloquea o desbloquea a otro hallazgo? |
| --- | --- | --- | --- | --- | --- |
| W7 - El formulario no tiene type=email, required, maxLength ni <label> (accesibilidad) | Comentarios.jsx:78-85 | Daño 2: inconsistencia o riesgo latente/ Lo nota el usuario oteniendo error con datos incorrectos en el formulario. | 1 | Sí | A ninguno |
| W9 - El botón no se desactiva al enviar, así que un doble clic duplica el comentario | Comentarios.jsx:28-48, :86 | Daño 2: inconsistencia o riesgo latente. / Posible duplicación de datos | 1 | No (los duplicados ya guardados) | A ninguno |
| W6a - Si la carga falla, muestra «0 comentarios» sin avisar | Comentarios.jsx:20, :25 | Daño 2: inconsistencia o riesgo latente. | 1 | Sí | A ninguno |
| A-T - Hay un solo test, solo del camino feliz, que afirma un único campo | comentarios.spec.ts:15-43 (:41-42), guard-tests.mjs:32 | Daño 3 - Contrato roto. Rompe cobertura / TDD | 2 | Sí | Todos los cambios de backend (Ax, Dx) |
| W4 - key={index} en las listas | Comentarios.jsx:57, :66 | Daño 1 - Cosmético | 1 | Sí | A ninguno |
| W2 - Pide el post dos veces solo para sacar los comentarios | Comentarios.jsx:17-26, DetallePost.jsx:123-134 | Daño 1 - Cosmético | 2 | Sí | Desbloquea W1 |
| W10 - La URL de la API está fija en el código | Comentarios.jsx:4, DetallePost.jsx:98 | Daño 1 - Cosmético | 2 | Sí | A ninguno |
| A8 - El POST devuelve el último comentario de la tabla, no el que acaba de crear; con dos envíos a la vez puede mostrar el email de otra persona | routes.ts:329-331 | Daño 4 - dato incorrecto visible para todo el mundo. Lo ve únicamente el usuario afectado. | 1 | Sí | A ninguno |
| D1 - estado es texto libre y conviven 'aprobado', 'Aprobado' y 'ok'; los comentarios aprobados 3, 4 y 9 no se ven | …comentarios_table.ts:18, blog_seeder.ts:510, :522, :575 | Daño 4 - dato incorrecto visible para todo el mundo. | 2 | No (al normalizar se pierde el valor original) | abarata A10, W1 |
| D2 - post_id no tiene clave foránea ni índice | …comentarios_table.ts:14 | Daño 2: inconsistencia o riesgo latente. | 1 | No (hay que limpiar huérfanos) | A ninguno |
| D3 - padre_id no tiene clave foránea | …comentarios_table.ts:20 | Daño 2: inconsistencia o riesgo latente. | 1 | No, (hay que limpiar filas) | A9, A13 |
| A3 - La clave de moderación está escrita en el código y publicada en el repo: cualquiera puede borrar cualquier comentario | routes.ts:25, :349-351 | Daño 5 - Seguridad o privacidad | 4 | No (sigue en el historial de git y lo borrado no vuelve) | A ninguno |
| D4 - creado_en admite nulos y se guarda sin zona horaria | …comentarios_table.ts:19, routes.ts:323 | Daño 1 - Cosmético | 2 | No (las fechas ambiguas no se pueden recuperar) | abarata W8 |
| A-ARQ - Toda la feature vive en las rutas: SQL, validación, reglas y respuesta | routes.ts:150-155, :295-334, :339-361, :366-393 | Daño 2: inconsistencia o riesgo latente. | 8 | Sí | abarata A1, A2, A4, A5, A6, A7, A9, A10, A13 y A16 |
| A10 - El recuento de comentarios cambia según el endpoint: el post 1 muestra 4 en la portada y 2 en el artículo | routes.ts:75-76, :223 frente a :153 | Daño 3 - Contrato roto | 2 | Sí | W1 |
| A13 - Al borrar un comentario padre, sus respuestas quedan huérfanas e invisibles | routes.ts:359, Comentarios.jsx:50 | Daño 3 - Contrato roto | 2 | No | A ninguno |
| A1 - El detalle público expone el email de quien comenta, además de estado y post_id; POST y PATCH también lo devuelven | routes.ts:152-155, :171, :329-333, :389-392 | Daño 5 - Seguridad o privacidad | 4 | Sí | reduce el riesgo de A2 |
| A2 - Para borrar o editar basta con conocer el email | routes.ts:350-353, :378-379 | Daño 5 - Seguridad o privacidad | 4 | No (las filas existentes no tienen un dueño verificable) | A4 |
| A5 - El POST responde a los errores de 4 formas distintas, una de ellas con un 200 | routes.ts:310-321 | Daño 3 - Contrato roto | 3 | Sí | A6, A7, A9 y W6b |
| W8 - String(creado_en).substring(0,10) puede pintar «null» y desfasar el día | Comentarios.jsx:59, :68 | Daño 1 - Cosmético | 1 | Sí | A ninguno |
| A16 - Cada ruta improvisa sus errores en lugar de centralizarlos en el handler | routes.ts:301, :344, :356, :371, :380, handler.ts:15-17 | Daño 1 - Cosmético | 6 | Sí | A ninguno |
| A6 - El POST valida lo mínimo: un @, 5 caracteres sin quitar espacios, sin máximos ni tipos | routes.ts:305-307, :316, :319 | Daño 2: inconsistencia o riesgo latente. | 1 | Sí | A ninguno |
| A7 - El PATCH acepta un cuerpo de 1 carácter, aunque el POST exige 5 | routes.ts:383-386 | Daño 2: inconsistencia o riesgo latente. | 1 | Sí | A ninguno |
| W6b - El éxito y el error se deducen de cuatro formas de respuesta distintas | Comentarios.jsx:37-44 | Daño 2: inconsistencia o riesgo latente. | 1 | Sí | A ninguno |
| A9 - padre_id no se valida: ni tipo, ni que exista, ni que sea del mismo post o esté aprobado | routes.ts:308, :326 | Daño 3 - Contrato roto | 2 | Sí | W3 |
| W1 - Dos recuentos independientes que cuentan también las respuestas y suman los comentarios pendientes | DetallePost.jsx:121,129,162, Comentarios.jsx:11,23,39,54 | Daño 3 - Contrato roto | 2 | Sí | A ninguno |
| A4 - DELETE y PATCH comprueban la autoría con reglas distintas (mayúsculas, moderador, solo pendientes) | routes.ts:352-353 frente a :379 | Daño 2: inconsistencia o riesgo latente. | 2 | Sí (no, si se normalizan los emails ya guardados) | A ninguno |
| W3 - Solo se pinta un nivel de respuestas, así que las de segundo nivel o con el padre oculto no se ven nunca | Comentarios.jsx:50, :63-64 | Daño 2: inconsistencia o riesgo latente. | 1 | Sí | A ninguno |


# Parte B: 

## 1. Cuántos hallazgos devolvió el agente, y cuántos comprobaste tú abriendo el archivo

el agente devolvió: 28
yo comprobé: 5 (reloj 45 min)

## 2. El hallazgo del que no supiste decidir la prioridad

A-T - Hay un solo test, solo del camino feliz, que afirma un único campo

El agente lo clasifica como prioritario por precedencia sore todo el backend, y para cumplir la norma de TDD.

Esta clasificación hace retrasar por precedencia todo lo relacionado con backend (incluyendo hallazos con daño significativo y con radio pequeño)


## 3. Algo que el agente afirmó y no pudiste verificar.

¿23 hallazgos no confirmados?

Daño estimado por el agente. Solo validado para hallazgos comprobados.

Si hay registros afectados por los hallazgos (si es necesaria migración de datos para corregir, y si será viable un revert o no en función de los datos que haya al aplicar cambios).
