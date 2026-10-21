# Auditoría priorizada — feature de comentarios

Alcance: crear, editar, borrar y listar comentarios en API (`backend/start/routes.ts`), base de
datos (migración 005 y semilla) y web (`Comentarios.jsx`). 

Base: `s6/start`, commit `a87a414`.

Orden: precedencia primero (lo que desbloquea va antes) y, dentro de cada nivel, daño / radio.
Daño: 3 = expone o destruye hoy; 2 = el lector ve algo mal; 1 = nadie todavía. Radio = sitios
de la columna 4. El daño máximo con precedencia mínima se parte en contención (C) y fondo (F).

Limitada a 10 filas; el resto va debajo de la tabla.

| Qué es | Dónde está | Qué se rompe, y quién lo nota | Sitios | ¿Revert? | ¿Bloquea / desbloquea? |
|---|---|---|---|---|---|
| **C1** · El detalle publica el correo de cada comentarista, y ese correo basta para borrar el comentario. | `routes.ts:153` `select * from comentarios` → `:171`; DELETE compara el correo en `:352-353`. | 3: Cualquiera lee los correos (regla 6) y borra comentarios ajenos: `DELETE /comentarios/2` con el correo publicado → 200. | 2: `routes.ts:153` y un test nuevo (hoy no hay ninguno). | Sí; los correos ya servidos no vuelven. | Contención de F1. |
| **C2** · La clave de moderación es un literal en el código. | `routes.ts:25` `SECRETO_SESION = 'dev-secret-cambiar'`, usada en `:351`; el `TODO` de `:22` lo admite. | 3: Quien lea el repositorio borra cualquier comentario, también los pendientes. | 5: `routes.ts:25`, `start/env.ts`, `.env.example`, `.env.test` y un test nuevo (hoy no hay ninguno). | Sí el código; la clave sigue en el historial, hay que rotarla. | Contención de F2. |
| **H1** · La feature tiene un único test, y afirma un solo campo. | `comentarios.spec.ts:41-42` `assertStatus(201)` + `autor_nombre`. | 1: Nadie todavía; quien refactorice no tiene red. | 1: `comentarios.spec.ts`. | Sí. | Desbloquea H5 y, a través de él, F1 y F2. |
| **H2** · `estado` es texto libre y conviven `'aprobado'`, `'Aprobado'` y `'ok'`. | Migración 005 `:18`, sin CHECK; semilla `:510`, `:522`, `:575`. | 2: El lector no ve 3 comentarios aprobados, y el listado cuenta 4 donde el detalle muestra 2 (comprobado contra la API, post 1). | 4: 1 migración y 3 líneas de la semilla. | No: el UPDATE pisa los valores originales. | Desbloquea H6 y H8 (abajo). |
| **H4** · La validación son `if`s a mano con cuatro contratos de error distintos. | `routes.ts:310-321`: `400 {error}`, `422 {message}`, `400 {errores}`, `200 {ok:false}`. | 2: Con un correo sin `@`, el lector ve «No se ha podido enviar.» (`Comentarios.jsx:44` no lee `errores`). | 7: `routes.ts:310`, `:313`, `:316`, `:319`, `:384`, `Comentarios.jsx:44` y 1 validador. | Sí, pero cambia el contrato de error. | Desbloquea H12–H15 y H17 (abajo). |
| **H5** · La regla de quién puede tocar un comentario está escrita dos veces y sale distinta. | DELETE `routes.ts:351-353` (clave, minúsculas) frente a PATCH `:379` (sin clave, exacta, solo pendientes). | 1: Nadie todavía: no hay interfaz de edición. | 2: `:351-353` y `:379`, a una función de servicio. | Sí. | Bloqueado por H1; desbloquea F1 y F2. |
| **H7** · La web suma 1 al recuento al enviar un comentario que queda pendiente. | `Comentarios.jsx:39` `setTotal(total + 1)`. | 2: Quien comenta ve «3 comentarios» con 2 en pantalla. | 1: `Comentarios.jsx:39`. | Sí. | A ninguno. |
| **H9** · POST devuelve el último comentario de la tabla, no el insertado. | `routes.ts:329-331` `order by id desc limit 1`. | 1: Nadie todavía; con envíos simultáneos, uno recibiría el correo de otro. | 1: `routes.ts:329-331`. | Sí. | A ninguno. |
| **F2** · La moderación usa una clave compartida en vez de usuarios autenticados. | `routes.ts:349-351`; la autenticación existe (`:416`), pero `users` no tiene rol. | 3: Quien tenga la clave borra todo, sin rastro, y no se puede revocar por persona. | 4: migración de rol, rutas con `middleware.auth()`, servicio de H5 y quitar la clave. | No: la migración no se deshace con un revert. | Bloqueado por H1 y H5. |
| **F1** · Saber el correo de alguien basta para editar o borrar sus comentarios. | `routes.ts:352-353` (DELETE) y `:379` (PATCH). | 3: Lo que C1 contiene: suplantar a cualquier comentarista. |  |  | Bloqueado por H1 y H5. |

Niveles (daño / radio): 

- **N0** C1 3/2 = 1,5 · C2 3/5 = 0,6
- **N1** H1 1/1 = 1,0 · H2 2/4 = 0,5 · H4 2/7 = 0,29
- **N2** H5 1/2 = 0,5
- **N3** H7 2/1 = 2,0 · H9 1/1 = 1,0
- **N4** F2 3/4 = 0,75 · F1 3/? (sin radio contado, queda la última)

## Fuera del límite de 10

Encontrados pero sin fila: 

- H3, filas crudas sin transformer (`routes.ts:171`, `:333`, `:392`) 
- H6, el detalle filtra `'aprobado'` exacto (`:153`) 
- H8, los recuentos usan `<> 'pendiente'` (`:76`,`:223`)
- H10, DELETE deja respuestas huérfanas (`:359`) 
- H11, `.catch(() => {})`(`Comentarios.jsx:25`) 
- H12, PATCH sin mínimo (`:384`) 
- H13, sin longitud máxima (`:319`) 
- H14, el correo solo se valida con `@` (`:316`) 
- H15, el mínimo cuenta espacios (`:319`) 
- H16, `post_id` sin FK (migración 005 `:14`) 
- H17, `padre_id` sin validar (`:308`) 
- H18, dos recuentos en la web (`DetallePost.jsx:30`, `Comentarios.jsx:11`) 
- H19, no hay forma de aprobar por API (`:388`) 
- H20, no hay interfaz para editar ni borrar (`Comentarios.jsx:30-34`).

## Propuestas sin medir

- Índice en `comentarios.post_id` → `EXPLAIN QUERY PLAN` y latencia de `routes.ts:75`, `:153` y `:223`.
- Quitar la subconsulta de recuento (`:75-76`, `:222-225`) → tiempo de `GET /posts` y de los relacionados bajo carga.
- Que `Comentarios.jsx:19` no vuelva a pedir el post → peticiones y bytes por vista de artículo.
- Paginar los comentarios → comentarios por post (p95 y máximo).
- Antispam en POST → envíos por IP y minuto, y proporción de spam entre los pendientes.
- Borrado lógico (`:359`) → cuántas veces se pide recuperar un comentario.
- Tope de longitud (H13) → p99 de la longitud de los comentarios reales.

## Fuera de alcance

- **Repositorio:** nada impide commitear secretos (el único hook es `guard-tests.mjs`). Propuesta:
  gitleaks como hook `PreToolUse`, como `pre-commit` de git y en CI.
- **Posts:** `GET /posts/:slug` expone el correo del autor (`routes.ts:111` → `:167`), y el cuerpo
  se inyecta sin sanear (`DetallePost.jsx:92`).

---

## Parte B — las tres líneas

1. **Hallazgos que devolvió el agente:** Inicialmente 24 filas, que son 22 hallazgos (C1/F1 y C2/F2 son el
   mismo hallazgo partido), más 7 propuestas sin medir fuera de la tabla. Posteriormente se redujo la tabla por falta de tiempo y para cumplir los requisito y quedó en 10, quedando las restantes 14 en la sección "Fuera del límite de 10". **Comprobados por mí
   abriendo el archivo:** 5: H1, C1, C2, F2, H9
2. **El hallazgo del que no supe decidir la prioridad:** H2 y H4, porque `CLAUDE.md` obliga a
   caracterizar un bug antes de corregirlo, lo que los haría depender de H1, pero esos tests se
   borrarían nada más crearse y no sé cuál de los dos criterios manda.
3. **Algo que el agente afirmó y no pude verificar:** H9, que con envíos simultáneos uno recibe
   el comentario y el correo de otro. Lancé 50 POST en paralelo, los 50 recibieron el suyo, y eso
   no basta para confirmarlo ni para descartarlo.
