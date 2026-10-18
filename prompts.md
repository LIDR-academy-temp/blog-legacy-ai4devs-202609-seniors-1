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

**Modelo:** Sonnet 5.5 High
**Herramienta:** Claude Code

```
Vas a auditar una sola feature de este blog: los comentarios. Una feature es una
funcionalidad de punta a punta, así que cuenta lo que forma parte de los comentarios
(crear, editar, borrar y listar) en sus tres capas:
- API: las rutas, controladores, validaciones y lógica de comentarios (backend/).
- Base de datos: la tabla de comentarios, sus migraciones y semillas.
- Web: lo que la web enseña y hace con los comentarios (frontend/): componentes,
  llamadas a la API y estados de error.

Solo esa feature. No audites posts, autores, categorías ni etiquetas. Si un hallazgo
de comentarios depende de alguna de ellas, lo mencionas en la última casilla, pero no
abras hallazgos sobre ellas.

Es una auditoría: no propongas soluciones ni modifiques código.

Escribe el resultado en docs/auditoria/comentarios.md (el directorio ya existe). Ese
archivo debe contener solo una tabla de Markdown, sin otras secciones salvo la lista
"Sin medir" que se describe más abajo. Una fila por hallazgo, con exactamente estas
seis columnas, en este orden:

1. Qué es: una sola frase. Si necesitas varias, son dos hallazgos. Empieza con un
   identificador (H1, H2...) para poder referirse a él.
2. Dónde está: archivo:línea y, entre comillas, la línea literal que lo demuestra.
   Si no puedes citar una línea concreta que lo demuestre, no escribas la fila.
3. Qué se rompe y quién lo nota: el efecto concreto y a quién le llega (quien usa el
   blog, quien lo mantiene, o "nadie todavía" si hoy no lo nota nadie).
4. Cuántos sitios hay que tocar: un número contado, no estimado, seguido entre
   paréntesis del comando de búsqueda con el que lo contaste.
5. ¿Se deshace con un revert?: sí o no, y si es que no, por qué (por ejemplo, porque
   afecta a datos o al esquema de la base de datos, o cambia lo que devuelve la API).
6. ¿Bloquea o desbloquea a otro hallazgo?: nombra al otro por su identificador, o
   escribe "a ninguno".

Reglas:
- Cada hallazgo debe poder comprobarse abriendo el archivo y buscando la línea que
  citas. Copia la línea tal cual, sin parafrasearla.
- Si no puedes determinar una casilla con lo que has leído, escribe "no determinado"
  y, entre paréntesis, qué te falta. No rellenes casillas por suposición.
- No ordenes la tabla todavía: déjala en el orden en que encuentres los hallazgos.
- Si tienes sospechas que no has podido medir (rendimiento, carga, comportamiento con
  muchos datos...), no las metas en la tabla: ponlas debajo en una lista "Sin medir",
  con qué habría que medir para confirmarlas.

Restricciones:
- Solo lectura sobre el proyecto. Puedes hacer peticiones GET a la API local
  (http://localhost:3401) para comprobar comportamiento; no hagas POST, PUT, PATCH ni
  DELETE.
- No ejecutes make test ni make db: regeneran archivos o modifican los datos.
- No modifiques ningún archivo salvo docs/auditoria/comentarios.md.
- No ejecutes git add, commit, checkout, switch, stash ni reset.
- Al terminar, responde solo con una línea: cuántos hallazgos has escrito, cuántos
  hay en "Sin medir" y la ruta del archivo.
```

**Qué salió:** 

31 hallazgos, 6 en "Sin medir", en docs/auditoria/comentarios.md.

---

## Prompt 2

**Modelo:** Sonnet 5.5 High
**Herramienta:** Claude Code

```
Hay un archivo de auditoría, docs/auditoria/comentarios.md, sobre la feature de comentarios de
este blog. Léelo. Tiene una tabla "Comprobadas por mí" con 6 hallazgos ya verificados a mano.

Propón en qué orden arreglarías esos 6 hallazgos, de lo que harías primero a lo que harías
último. Elige tú el criterio con el que ordenarlos.

Respóndeme en el chat con:
1. El orden final, en una lista numerada.
2. Para cada hallazgo, una línea con por qué va en esa posición.
3. El criterio que has seguido, en una o dos frases.

Restricciones:
- No modifiques ningún archivo, tampoco docs/auditoria/comentarios.md: solo quiero tu
  propuesta en el chat.
- No ejecutes git add, commit, checkout, switch, stash ni reset.
```

**Qué salió:** 

Orden propuesto

1. H3: el detalle del post publica el correo de quien comenta
2. H11: la clave de moderación está escrita en el código
3. H10: DELETE acepta solo el correo del autor
4. H16: estado es texto libre y ya hay valores no canónicos
5. H2: el detalle oculta los comentarios Aprobado y ok
6. H1: el recuento del listado no coincide con el del detalle

Por qué va en cada posición

1. H3: hoy mismo cualquiera, sin autenticar, obtiene correos de personas por un GET que he comprobado. Además ese correo es la credencial que acepta H10, así que cerrarlo primero reduce el daño de H10 desde ya y no depende de ningún otro hallazgo.
2. H11: permite borrar cualquier comentario a quien tenga el repo, y es de los pocos arreglos baratos. El valor ya está en el historial de git, así que cuanto antes se cambie, antes deja de valer. Lo pongo antes que H10 por el coste, pero no sé si el repo es público. Si lo fuera, subiría.
3. H10: es el más grave de los tres en alcance (cualquiera que conozca un correo). Va después porque, una vez cerrado H3, ya no se explota tan fácilmente, y porque arreglarlo exige decidir qué credencial sustituye al correo, que es una decisión de diseño y no un cambio de una línea.
4. H16: es la causa raíz de H1 y H2, y su arreglo toca esquema y datos, con lo que un revert no lo deshace. Prefiero hacerlo con calma y antes que los dos que dependen de él, no entre medias de ellos.
5. H2: es el que más daño causa a quien lee y a quien comentó, porque 3 comentarios aprobados no se ven. Va después de H16 porque la regla de qué es visible depende de qué valores de estado son válidos.
6. H1: es solo una cifra desalineada, sin pérdida de contenido. Va el último porque debe contar exactamente lo que H2 ya decidió mostrar. Si se hace antes, habría que volver a tocarlo.

Criterio: primero lo que expone o permite destruir datos de terceros ahora mismo (la gravedad manda), y dentro de cada grupo respeto las dependencias de la propia tabla (H16 antes de H2 y H1; H3 antes de H10). Todo lo que cambia lo que devuelve la API necesita antes tests de caracterización (H31, fuera de estos seis), porque hoy solo existe un test.

---

## Prompt 3

**Modelo:** Sonnet 5.5 High
**Herramienta:** Claude Code

```
Te cuento cómo he decidido ordenar los hallazgos comprobados de docs/auditoria/comentarios.md y por qué. No tienes que modificar nada ni hacer ninguna tarea: quiero que lo tengas en cuenta y que quede registrado. Responde solo con "Anotado".

Mi orden final es:
1. H31 (solo hay un test y cubre únicamente el camino feliz del alta)
2. H3a (contención de H3: dejar de enviar el correo en el detalle público del post)
3. H16 (el campo estado es texto libre)
4. H10 (el borrado se autoriza solo con el correo del autor)
5. H11 (clave de moderación escrita en el código)
6. H2 (el detalle oculta los comentarios guardados como Aprobado y ok)
7. H1 (el recuento del listado no coincide con el del detalle)
8. H3b (arreglo de fondo de H3: los otros dos sitios que devuelven la fila entera)

Por qué lo he ordenado así:
- He aplicado la precedencia primero, y dentro de cada nivel el daño dividido por el radio (cuántos sitios hay que tocar). Tu propuesta ordenaba por gravedad; yo no.
- H31 va primero porque casi cualquier arreglo cambia lo que devuelve la API y, con un solo test del camino feliz, no habría red para detectar qué se rompe. El CLAUDE.md del blog pide caracterizar el
  comportamiento actual antes de cambiarlo.
- H3 lo he partido en dos porque es el hallazgo de más daño (cualquiera, sin autenticar, obtiene el correo de quien comenta, lo he comprobado con curl) pero su arreglo de fondo tiene baja
  precedencia: depende de H31 y toca más sitios. La contención (H3a) es un cambio de código en un solo sitio, la línea 153, y corta el daño hoy. El arreglo de fondo (H3b) cubre los otros dos sitios
  y espera su turno.
- Dudé si poner H3a antes de H31, porque corta el daño más alto, pero cambia lo que devuelve la API y eso es justo lo que H31 dice que no se debe hacer sin tests. Lo urgente frente a lo que bloquea; me he quedado con H31 primero.
- H16 va tercero porque es el único comprobado que desbloquea a otros (H1 y H2): hay que decidir qué valores de estado son válidos antes de arreglar los filtros. Toca esquema y datos y un revert no lo deshace.
- H10 va antes que H11: las dos tocan 2 sitios, así que decide el daño. H10 está abierto a cualquiera que conozca un correo; H11 solo a quien tenga el repo, y no sé si es público. Además el radio de H10 (2 sitios) no recoge su coste real, porque hay que decidir qué credencial sustituye al correo.
- H2 y H1 van después de H16. H2 oculta comentarios aprobados, que es pérdida de contenido; H1 es solo una cifra, y debe contar lo que H2 decida mostrar.

En el archivo, H3 aparece ya partido en H3a y H3b, y las filas están en este orden. No cambies nada.
```

**Qué salió:** 

Anotado