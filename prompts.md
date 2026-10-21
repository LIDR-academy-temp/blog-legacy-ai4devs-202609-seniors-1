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

---

## Prompt 1

**Modelo:** Claude Sonnet 5
**Herramienta:** Claude Code

```
Genérame la auditoria priorizada de la feature de comentarios en esta aplicación que es un BLOG:
- Déjala escrita en el documento docs/auditoria/comentarios.md, NO en el chat.
- Ten en cuenta cualquier aspecto relacionado con la funcionalidad de comentarios en todas las capas de la API, tanto a nivel de backend como frontend.
  - Sus rutas API
  - La tabla de base de datos
  - Todo lo que se muestra en la web al respeto.
- Hay otras funcionalidades. NO me hagas la auditoría de estas.
- Utiliza el siguiente criterio de priorización:
	- Primero la precedencia: lo que simplifica, abarata o desbloquea otro punto va antes.
	- Luego el daño dividido por el radio
		- Como daño se entiende, ¿que se rompe y quién o qué lo nota?. Indícalo explícitamente, y si no encuentras nada, índicalo explícitamente con "Nadie todavía".
		- Radio: En cuantos sitios se tiene que tocar el cambio. Cuéntalos e indica el fichero y la línea donde está.

- El formato del fichero debe ser este OBLIGATORIAMENTE, no es negociable:


El formato no es negociable: una fila por hallazgo, con estas seis casillas.

- Qué es. Una frase. Si necesitas tres, es que hay dos hallazgos.

- Dónde está, con la línea que lo demuestra. Sin esta casilla, la fila no se escribe.

- Qué se rompe, y quién lo nota. El efecto concreto y a quién le llega. "Nadie todavía" es una respuesta válida y buena.

- Cuántos sitios hay que tocar. Contados, no estimados.

- ¿Se deshace con un revert? Sí o no, y si es que no, por qué no.

- ¿Bloquea o desbloquea a otro hallazgo? Nombrando al otro. "A ninguno" es la respuesta más frecuente.



Si de alguna propuesta no puedes medir los datos la metes fuera de la tabla, con lo que habría que medir escrito al lado.

```

**Qué salió:**

```
He escrito la auditoría priorizada de la feature de comentarios en docs/auditoria/comentarios.md (7 hallazgos en la tabla obligatoria de seis casillas, más un punto fuera de tabla por falta de datos para medir el daño). Cubre: fuga del correo de quien comenta en la respuesta pública, el estado de moderación sin normalizar (que descoloca el recuento frente a lo que se ve), la clave de moderación hardcodeada, el recuento de comentarios triplicado y desincronizado entre backend y los dos componentes de React, el contrato de errores inconsistente del POST de comentarios, las respuestas huérfanas al borrar un comentario padre, y la autorización del PATCH que no coincide con la del DELETE.
```

## Prompt 2

**Modelo:** Claude Sonnet 5
**Herramienta:** Claude Code

```
 Puedes darme los argumentos para tildar el hallazgo 5 como el que no super decidir la prioridad según esta pregunta:
  2. El hallazgo del que no supiste decidir la prioridad, y en una frase, por qué. Lo que interesa es qué dos cosas se contradecían: lo grave contra lo que bloquea, lo urgente contra lo caro, lo que te pedían contra lo que veías.

```

**Qué salió:**

```
Hallazgo 5 fue el que más me costó priorizar: el conflicto es gravedad de diseño vs. impacto real hoy.

Por un lado, que un endpoint público devuelva 200 {ok:false} para un fallo de validación es grave en abstracto — rompe el contrato HTTP, afecta a 4 sitios distintos, y es exactamente el tipo de cosa que "parece que funciona" hasta que un cliente nuevo confía en el status code. Eso empuja a subirlo cerca de la cabecera del ranking (como el 2 o el 3).

Por otro lado, el daño real medido como "quién lo nota hoy" es prácticamente cero: el único cliente que existe (Comentarios.jsx:44) ya lo esquiva leyendo d.error || d.message || d.motivo, así que no bloquea nada, no urge, y nadie en la UI actual lo sufre. En eso se parece al 6 y al 7, que bajé por ser "nadie todavía".

La contradicción es esa: si prioritizo por lo grave que es el defecto en sí (inconsistencia de contrato en una ruta pública), sube; si prioritizo por lo urgente/caro que es dejarlo así hoy (nadie lo sufre, es barato posponerlo), baja. Los otros hallazgos no tenían ese empate — el 2 (fuga de email) y el 3 (secreto hardcodeado) son graves y tienen un afectado real ya identificable; el 6 y el 7 son graves pero sin ningún afectado ni radio alcanzable desde la UI. El 5 queda justo en medio: grave como el 2/3, pero sin afectado real como el 6/7, y no hay una regla en el criterio que pedido ("precedencia, luego daño/radio") que resuelva ese empate — el criterio asume que daño y urgencia van juntos, y aquí no iban.
```

## Prompt 3

**Modelo:** Claude Sonnet 5
**Herramienta:** Claude Code

```
Pero realmente sólo afecta a 1 sitio, la línea que devuelve el 200 cuando es demasiado corto.
```

**Qué salió:**

```
Tienes razón — mezclé dos cosas en una fila, y el formato que pediste es explícito en eso: "si necesitas tres frases para 'qué es', es que hay dos hallazgos". Los 4 casos no tienen la misma gravedad:

- Nombre vacío (400), email vacío (422), email sin "@" (400): las tres son respuestas de fallo con código 4xx. Son inconsistentes entre sí (formas de cuerpo distintas, 400 vs 422), pero un cliente que solo mira el status code ya detecta el fallo en los tres casos. Eso es una inconsistencia cosmética, no una rotura.
- Cuerpo corto (línea 320): responde 200 con {ok:false}. Ahí sí hay rotura real — cualquier cliente que confíe en el status code (no en el cuerpo) se cree que el envío tuvo éxito cuando no lo tuvo. Y está en un único sitio: backend/start/routes.ts:320.

Voy a partir el hallazgo 5 en dos filas y corregir el radio.

● Update(docs/auditoria/comentarios.md)
Added 2 lines, removed 1 line

Esto también cambia el orden: con radio 1 y daño real (rompe la semántica HTTP para cualquier integrador), el caso del 200 sube cerca del hallazgo de la fuga de email, no se queda a medias como pensaba antes. Voy a renumerar la tabla entera para que el orden siga reflejando el criterio.

● Update(docs/auditoria/comentarios.md)
Added 6 lines, removed 6 lines

He corregido el radio (era 1, no 4) y partido el hallazgo en dos filas, porque mezclaba una rotura real (200 en un fallo) con una inconsistencia cosmética (400 vs 422 con distinta forma de cuerpo), que es justo el caso que la plantilla pide separar. Con el radio corregido, el caso del 200 sube al puesto 4, junto a la fuga de email y la clave hardcodeada, en vez de quedarse empatado a medias como dije antes. La tabla ahora tiene 8 filas numeradas en el orden de prioridad corregido.
```

## Prompt 4

**Modelo:** Claude Sonnet 5
**Herramienta:** Claude Code

```
Me puedes reescribir la respuesta de
  El hallazgo del que no supiste decidir la prioridad, y en una frase, por qué. Lo que interesa es qué dos cosas se contradecían: lo grave contra lo que bloquea, lo urgente contra lo caro, lo que te pedían contra lo que veías.

  para elque ahora es el hallazgo 4
```

**Qué salió:**

```
No era solo un problema de radio: incluso corregido a 1, la tensión de fondo sigue viva en el hallazgo 4, solo que cambia de forma. Te la reescribo:

---

El hallazgo del que no supe decidir la prioridad: el 4 (POST /posts/:slug/comentarios responde 200 cuando el comentario es demasiado corto), porque enfrentaba lo grave en el contrato contra lo irrelevante en la práctica actual.

Por un lado, es grave por definición: una ruta pública que devuelve éxito (200) ante un fallo rompe la regla más básica de un contrato HTTP, y es el tipo de defecto que un test de contrato cazaría al instante. Eso pide tratarlo como prioritario.

Por otro lado, hoy no lo nota nadie: el único cliente que existe (Comentarios.jsx:44) ya ignora el status code y lee el cuerpo directamente, así que el defecto no bloquea nada ni afecta a ningún usuario real ahora mismo — en eso se parece a los hallazgos 7 y 8, que bajé precisamente por ser "nadie todavía".

Lo que rompió el empate fue corregir el radio: al verificar que el fallo real vive en una sola línea (routes.ts:320), el coste de arreglarlo deja de pesar en la decisión — es barato independientemente de cuánto importe hoy. Así que acabé priorizándolo alto no por urgencia (que es cero) sino porque la combinación "grave + gratis de arreglar" no tenía motivo para esperar; si hubiera costado tocar 4 sitios, como pensé al principio, lo habría dejado más abajo, cerca del 6, a la espera de que alguien lo sufriera primero.
```


## Prompt 5

**Modelo:** Claude Sonnet 5
**Herramienta:** Claude Code

```
Puedes redactarme la respuesta a la pregunta
  3. Algo que el agente afirmó y no pudiste verificar. Ojo al matiz: no es lo que dijo mal. Es lo que no pudiste comprobar ni a favor ni en contra, y te tocó dejar en el aire.

  Para el hallazgo que ha quedado fuera de la tabla.
  ```

**Qué salió:**

```
Algo que afirmé y no se pudo verificar: que la falta de índice en comentarios.post_id (hallazgo fuera de tabla) es un problema real de rendimiento y no solo deuda cosmética.

Afirmé que, al no haber índice, cada lectura de comentarios por post (routes.ts:152-154 y :222-225) hace un recorrido completo de la tabla, y que eso "escala linealmente" con el volumen. Es una afirmación plausible sobre cómo funciona SQLite sin índice, pero no es algo que haya comprobado: con los 9 comentarios de la semilla no hay manera de medir una diferencia real, y no tengo un volumen de datos representativo (ni una forma de generarlo y lanzar la query contra better-sqlite3) para confirmar en qué punto ese recorrido empieza a notarse de verdad.

Así que quedó deliberadamente fuera de la tabla, con el matiz explícito de que habría que medirlo. No es que crea que esté mal dicho — es que no pude comprobarlo ni a favor ni en contra, y no quise colarlo en la tabla con un "cuántos sitios" o un "qué se rompe" inventados solo para rellenar la casilla.


```
