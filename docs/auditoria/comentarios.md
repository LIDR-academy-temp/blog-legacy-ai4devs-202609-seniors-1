# Auditoría priorizada — feature de comentarios

> Alcance: **solo comentarios** (crear, editar, borrar, listar) en sus tres capas: rutas de la API,
> tabla `comentarios` y lo que la web enseña. Nada de posts, autores, categorías ni etiquetas.
>
> Punto de partida anotado antes de tocar nada: **`make test` → 1 passed (1)**.
> Toda la feature vive en `backend/start/routes.ts` con SQL crudo: no hay controladores ni modelos.

---

## La tabla

**Orden: precedencia primero; dentro de cada nivel, daño ÷ radio.**

| # | Qué es | Dónde está | Qué se rompe, y quién lo nota | Sitios a tocar | ¿Revert? | ¿Bloquea / desbloquea? |
|:-:|---|---|---|:-:|:-:|---|
| **1** | El `select *` sobre `comentarios` devuelve **también `autor_email`**, y sale por la respuesta | `routes.ts:153` (detalle del post) · `:330` (alta) · `:389` (edición) | **El correo de quien comenta llega a cualquier visitante del blog.** Medido: `make fuga` saca 3 correos reales de un post público | **3** *(las tres respuestas que devuelven filas de comentario al cliente)* | **Sí**, limpio | Comparte la línea 153 con el **#2**: **quien entre primero abarata al otro** |
| **2** | `estado` es **texto libre** y la comparación del listado es exacta | `routes.ts:153` · `:75` · `:223` · `:325` · `:379` | **Comentarios aprobados que no se ven.** Medido en la base: de **9** comentarios hay `aprobado`×4, `Aprobado`×2, `ok`×1, `pendiente`×2 → **3 aprobados invisibles** para el lector | **5** | **No.** Normalizar reescribe filas ya guardadas: entra con una persona delante | **Desbloquea el #3** (su regla depende de que `'pendiente'` signifique una sola cosa). Lo abarata el **#1** |
| **3** | La regla de **quién puede tocar su propio comentario está escrita dos veces, y distinta** | `routes.ts:353` (borrar) vs `:379` (editar) | **La misma persona puede borrar su comentario y no poder editarlo.** Borrar normaliza mayúsculas y espacios y admite la clave de moderación; editar compara el correo tal cual **y exige que siga `pendiente`**. **El moderador no puede editar nada** | **2** | **Sí**, limpio | **Depende del #2** |
| **4** | Tras insertar, el comentario nuevo se lee con **`order by id desc limit 1`**, sin filtrar por el que se acaba de crear | `routes.ts:330` | **Nadie todavía.** Con dos altas simultáneas devolvería el comentario de otra persona —**y su correo**—. Es un **riesgo**, no un daño: no he conseguido reproducirlo | **1** | **Sí**, limpio | A ninguno |
| **5** | El POST de alta responde con **cuatro formas de error distintas**, una de ellas **`200`** | `routes.ts:310` `400 {error}` · `:313` `422 {message}` · `:316` `400 {errores:[]}` · `:320` **`200 {ok:false}`** | **Nadie todavía:** comprobado que el frontend (`Comentarios.jsx:44`) lee las cuatro (`d.error \|\| d.message \|\| d.motivo`). **Riesgo para el siguiente consumidor**, que verá un `200` en un fallo de validación | **1** | **Sí**, limpio | A ninguno |

> ⬜ **Casillas en blanco a propósito:** en el **#4** no pude rellenar «qué se rompe» con un efecto
> observado, porque **no logré reproducir la concurrencia en el tiempo del reloj**. Queda como
> riesgo derivado del código, no como daño medido.

### Fuera de la tabla — propuestas sin medir

| Propuesta | Por qué está fuera | Qué habría que medir |
|---|---|---|
| **«El conteo de comentarios hace una consulta por relacionado; habría que agruparlo»** (`routes.ts:217-229`, y el comentario del propio código lo dice) | **Es una opinión, no un hallazgo.** El código efectivamente consulta en bucle, **pero nadie ha medido que eso duela** con 8 posts y 9 comentarios | **Cuánto tarda `/posts` hoy y cuánto con la consulta agrupada**, con el volumen real. Si la diferencia no se nota, el cambio no entra |

---

## Las tres líneas

1. **El agente devolvió 6 candidatos; comprobé 4 abriendo el archivo y ejecutando.** El #1 con
   `make fuga` (3 correos en pantalla), el #2 consultando la base (4 valores distintos de `estado`),
   el #3 leyendo las dos reglas una al lado de otra, y el #5 con un `curl` que devolvió `200`. **El
   #4 no lo pude reproducir** y el sexto —el N+1— **no lo medí**, por eso está fuera de la tabla.

2. **No supe decidir entre el #1 y el #2, y lo que se contradecía era el daño contra lo que
   desbloquea:** la fuga tiene el daño más alto y medido —correos de terceros a cualquier
   visitante—, pero normalizar `estado` **toca la misma línea 153 y además desbloquea el #3**, así
   que ponerlo segundo significa volver a esa línea dos veces. **Dejé la fuga primero porque es
   dato personal saliendo hoy**, y no porque tenga claro que sea el orden barato.

3. **Lo que el agente afirmó y no pude verificar ni a favor ni en contra: que quitar `autor_email`
   de la respuesta no rompa nada.** Comprobé que el frontend de este repo **no lo lee** —no aparece
   en `Comentarios.jsx` salvo para enviarlo—, pero **el contrato de una API no vive solo en su
   repositorio**. Si hay un boletín, un script o un tercero leyendo ese campo, **desde aquí no hay
   forma de saberlo**, y la respuesta tendría que escribirla alguien que estuviera entonces.

---

## 🧭 Nota de método

Auditar **una sola feature** en vez del proyecto entero hizo lo que la lección anunciaba: el
`select *` apareció **una vez con su lista de tres sitios**, en vez de tres hallazgos sueltos. Y
los dos arreglos de fondo —la fuga y el `estado`— **caen en la misma línea**, que es algo que no se
ve si los hallazgos llegan de partes distintas del sistema.
