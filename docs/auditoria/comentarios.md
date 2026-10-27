# Auditoría de la feature de comentarios

Alcance: solo comentarios (crear, editar, borrar y listar), en la API, la base de datos y la web.
Rutas abreviadas: `routes.ts` es `backend/start/routes.ts`; `migración` es `backend/database/migrations/1769000000005_create_comentarios_table.ts`; `semilla` es `backend/database/seeders/blog_seeder.ts`; `Comentarios.jsx` y `DetallePost.jsx` están en `frontend/src/componentes/`.

Revisados por el usuario abriendo el archivo (según su declaración): C1, C2 y C3. El resto lo ha leído el agente y no lo ha revisado el usuario. Casilla en blanco: sin evidencia para rellenarla.

## Tabla

| Qué es | Dónde está | Qué se rompe y quién lo nota | Cuántos sitios | ¿Se deshace con un revert? | ¿Bloquea o desbloquea a otro? |
|---|---|---|---|---|---|
| C11 · Solo el alta feliz tiene prueba; editar, borrar, listar y validar no. | `backend/tests/functional/comentarios.spec.ts:15-43` (cabecera `:5-11`) | Nada hoy; quien cambie C1, C3, C4, C6 o C7 no tendría aviso. | | Sí, son pruebas. | Desbloquea C1, C3, C4, C6, C7 (`CLAUDE.md` pide caracterizar antes). |
| C1 · El detalle de un post (`select *`) y el alta y la edición devuelven el `autor_email` de cada comentario. | `routes.ts:152-155→171`, `:329-333`, `:389-392`; `migración:16` | Cualquier visitante ve los correos de quien comenta (GET en local, 3 posts). Producción: sin verificar. | 3 | El arreglo sí; los correos ya servidos no se recuperan. | Abarata C3. |
| C2a · Contención de C2: la clave de moderación escrita en el código debe dejar de valer. | `routes.ts:25` (`TODO(2024-11)`), `:349-351` | Quien lea el repo (público) podría borrar comentarios con `x-clave-moderacion`. Uso en producción y borrado real: sin verificar ni ejecutar. | 2 | No del todo: el valor ya está en el historial público; hay que rotarla. | No bloquea ni espera a nadie. |
| C6 · El recuento cuenta `estado <> 'pendiente'` y la lista `= 'aprobado'`; `estado` es texto libre (`Aprobado`, `ok`). | `routes.ts:75-76`, `:223` frente a `:153`; `migración:4-6,18`; `semilla:510,522,575` | El lector ve 4 en la lista y 2 en el detalle de un post, y 1 frente a 0 en otro (GET en local). Estados en datos reales: sin verificar. | 4, más los datos | Código y migración sí; normalizar datos no. | Desbloquea C5. |
| C3 · Borrar y editar se autorizan con el correo que escribe quien llama; no hay sesión. | `routes.ts:350-353`, `:378-379` | Con el correo de otro (C1 lo publica) se podría borrar o editar su comentario. Correo ajeno: 403 (comprobado); correo correcto: no ejecutado. Si `request.input` lee también la query: sin verificar. | 2 | Sí. | Se resuelve con C4; lo abarata C1. |
| C4 · Borrar y editar aplican reglas distintas (clave, mayúsculas y estado). | `routes.ts:352-353` frente a `:379` (lo admite `:375-377`) | Con otras mayúsculas en el correo se puede borrar pero no editar; un comentario aprobado no se corrige. Deducido del código, sin ejecutar. | 2 | Sí. | Se resuelve con C3. |
| C7 · El POST rechaza de 4 formas (400 `{error}`, 422 `{message}`, 400 `{errores}`, 200 `{ok:false}`); el PATCH no exige los 5 caracteres. | `routes.ts:311,314,317,320`; `:319` frente a `:384`; `Comentarios.jsx:44` | 4 rechazos comprobados en local (la tabla siguió en 9 filas); la web no lee `errores`. Editar a 1 carácter: deducido, sin ejecutar. | 6 | Sí, pero cambia el contrato de la API. | Lo desbloquea C11. |
| C5 · Los comentarios nuevos nacen `pendiente` y ninguna ruta ni pantalla del repo cambia el estado. | `routes.ts:326`, `:153`; único `update comentarios` `:388` (cuerpo); `Comentarios.jsx:38` | Quien comenta no ve nunca su comentario publicado. ¿Panel externo que modere? La semilla cita uno (`semilla:509`): sin verificar. | | Sí, en código. | Lo desbloquea C6; C2b espera a este. |
| C2b · Arreglo de fondo de C2: moderar sin depender de una clave compartida. | `routes.ts:25`, `:349-351` | Mientras no exista, otra filtración da de nuevo poder total. | | Sí, en código. | Espera a C3 y C5 (razonamiento del agente, no un dato del código). |

Orden propuesto por el agente, no definitivo. Se ordena por precedencia y, dentro de cada nivel, por daño entre radio (el radio es el número de la cuarta casilla). Niveles: 1) C11 y C2a, que no esperan a nadie; 2) C1, C6, C3, C4 y C7, que C11 desbloquea; 3) C5, que espera a C6; 4) C2b, que espera a C3 y C5. Lo que la evidencia no resuelve: el daño no está cuantificado, así que el cociente no se puede calcular y el orden dentro de cada nivel se apoya solo en el radio y en las dependencias; entre C11 y C2a, y entre C1 y C11, el conflicto sigue sin decidir (Parte B, línea 2). La prioridad final la decide el usuario.

## Fuera de la tabla: sin medir

- C8 · `routes.ts:329-331` devuelve el último comentario de la tabla, no el insertado (hipótesis de condición de carrera). Experimento: dos POST casi simultáneos sobre una copia desechable de la base, y ver si uno recibe el comentario del otro.
- C9 · `padre_id` sin validar y borrado sin cascada (`routes.ts:308,326,359`; `Comentarios.jsx:50,63-64`; `migración:4-7,20`). Hoy nadie lo nota en la web: el formulario no envía `padre_id` (`Comentarios.jsx:33`). Experimento, en una copia: crear una respuesta a un id inexistente y borrar un padre, y mirar qué muestra la web.
- C10 · El contador de la web sube al comentar aunque el comentario quede pendiente (`Comentarios.jsx:23,39`; `DetallePost.jsx:38,71`). Experimento, en una copia: comentar y comparar las dos cifras de la página.
- Tamaño máximo y anti-spam del POST. Experimento, en una copia: medir qué tamaño acepta el servidor y cuántos envíos por minuto tolera.

## Parte B: las tres líneas

1. Devueltos por el agente: 11 · Comprobados por mí abriendo el archivo: 3
2. Prioridad que no supe decidir: No supe decidir si priorizar C1 o C11: C1 expone correos en las respuestas de la API local y urge limitar los datos devueltos, pero C11 plantea la necesidad de tener pruebas de caracterización antes de modificar el comportamiento heredado.
3. Afirmación no verificada: El agente planteó que podría existir un panel de moderación fuera del repositorio, a partir de una referencia en la semilla de datos, pero no pude comprobar si ese panel existe ni si está conectado a esta API.
