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

**Modelo:** Opus 5.5
**Herramienta:** Claude Code

```
resume los últimos commits, y busca en qué archivos aparece la palabra autor
```

**Qué salió:** funcionó a la primera: resumen de los 3 commits y 16 archivos con «autor» (más en `blog_seeder.ts` y `routes.ts`).


## Prompt 2

**Modelo:** Opus 5.5
**Herramienta:** Claude Code

```
Genera una auditoría priorizada de la feature de comentarios y déjala escrita en docs/auditoria/comentarios.md. Este blog tiene cinco features del blog punta a punta (posts, comentarios, autores, categorías y etiquetas) pero sólo te deber centrar en los comentarios, no en el sistema entero. Céntrate en sus 3 capas (crear, editar, borrar y listar comentarios). El orden de los hallazgos deberían seguir primero el criterio de precedencia (lo que abarata o desbloquea a otro hallazgo va antes), y dentro de cada nivel, el daño dividido por el radio. Si un hallazgo tiene el daño más alto y la precedencia más baja a la vez, sepáralo en 2 filas, una de contención que corta el daño hoy y el arreglo de fondo que espera su turno. Toda propuesta que nadie haya medido va fuera de la tabla con lo que harbía que medir escrito la lado. Respecto al formato, crea una fila por hallazgo con estas 6 columnas: 

Qué es. Una frase. Si necesitas tres, es que hay dos hallazgos.

Dónde está, con la línea que lo demuestra. Sin esta casilla, la fila no se escribe.

Qué se rompe, y quién lo nota. El efecto concreto y a quién le llega. "Nadie todavía" es una respuesta válida y buena.

Cuántos sitios hay que tocar. Contados, no estimados.

¿Se deshace con un revert? Sí o no, y si es que no, por qué no.

¿Bloquea o desbloquea a otro hallazgo? Nombrando al otro. "A ninguno" es la respuesta más frecuente.
```

## Prompt 3

**Modelo:** Opus 5.5
**Herramienta:** Claude Code

```
He comprobado H1 corriendo los tests y mirando el fichero. Por qué H1 está tan arriba? Por qué desbloquea H2, H3, H4, H5? Ningún usuario nota que no haya tests, sólo el developer que refactoriza. Ah! Ya entiendo, porque abarata o reduce el riesgo de la refactorización
```

## Prompt 4

**Modelo:** Opus 5.5
**Herramienta:** Claude Code

```
Sí cambia en la tabla que H1 desbloquea a H3 y H5. Si escribimos tests para el comportamiento actual de H2 y H4 luego se tendrán que refactorizar los tests para los nuevos comportamientos y no tiene sentido, no es eficiente
```

## Prompt 5

**Modelo:** Opus 5.5
**Herramienta:** Claude Code

```
Ok añadelo en la Parte B linea 2. Segun CLAUDE.md estamos rompiendo regla de proyecto  «primero se caracteriza el bug, luego se decide si se corrige, y la corrección cambia el test de caracterización por uno nuevo». En base a ello deberíamos crear tests del código original primero, aunque en este caso los vayamos a borrar nada más crearlos porque se implementará H2 o H4. Por tanto, no sabría la prioridad aquí porque no sé si debo seguir CLAUDE.md o realizar la auditoría y en base al orden que define no crearlos tendría más sentido y no habría dependencia
```

## Prompt 6

**Modelo:** Opus 5.5
**Herramienta:** Claude Code

```
Comprobando C1. http://localhost:3401/posts/diez-tendencias-de-diseno-que-pueden-cambiar-la-web-moderna veo que estado es "publicado" no "aprobado". Entonces no deberia estar devolviendo los emails de los comentarios. Esta es la salida "

{
  "post": {
    "id": 1,
    "titulo": "Diez tendencias de diseño que pueden cambiar la web moderna",
    "slug": "diez-tendencias-de-diseno-que-pueden-cambiar-la-web-moderna",
    "resumen": "Cada enero alguien publica su lista y cada diciembre la mitad no ha pasado. Repasamos las diez que sí han dejado rastro en proyectos reales.",
    "cuerpo": "## La lista de siempre, con una vara de medir\n\nUna tendencia solo cuenta si alguien la ha llevado a producción y ha sobrevivido al primer rediseño. Con ese filtro, la lista de veinte se queda en diez.\n\n![Tablero con diez tarjetas de tendencias ordenadas por adopción](https://picsum.photos/seed/corriente-1a/1200/675)\n*Ordenadas por adopción real, no por cuántas veces aparecen en una charla.*\n\n## Uno: la retícula vuelve a verse\n\nDurante años la retícula era el andamio que se quitaba al final. Ahora se deja a la vista, con líneas finas y márgenes anchos, y funciona como señal de que alguien pensó la página antes de escribirla.\n\n## Dos: tipografía con trabajo dentro\n\nLas fuentes variables dejaron de ser un truco. Un solo archivo cubre seis pesos y el sitio carga menos que antes con dos.\n\n<p>Este párrafo viene con HTML tal cual desde el editor antiguo, porque el CMS lo permitía.</p>\n<img src=\"/no-existe.png\" onerror=\"console.warn('Demo de XSS: el cuerpo del post se inyecta sin sanear')\">\n\n## Tres: menos movimiento, mejor movimiento\n\nLa animación de entrada de cada bloque murió de éxito. Lo que queda es transición corta, con propósito, y respeto por `prefers-reduced-motion`.\n\n![Comparación entre una entrada animada larga y una transición corta](https://picsum.photos/seed/corriente-1b/1200/675)\n*A la derecha, la versión que nadie pide desactivar.*\n\n## Y las siete restantes\n\nModo oscuro con contraste medido, ilustración propia en vez de banco de imágenes, formularios de un solo campo por pantalla, buscador que entiende sinónimos, portadas sin texto encima, paletas de tres colores y componentes que documentan su propio uso.",
    "imagen_portada": "https://picsum.photos/seed/corriente-1/1200/630",
    "estado": "publicado",
    "publicado_en": "2026-03-14 09:00:00",
    "autor_id": 1,
    "categoria_id": 2,
    "autor_nombre": "Marta Ruiz Calvo",
    "autor_email": "marta.ruiz@corriente.es",
    "categoria_nombre": "Tendencias",
    "etiquetas": "Tendencias, Diseño web, Artículos",
    "created_at": "2026-03-10 12:00:00",
    "updated_at": "2026-03-14 09:00:00"
  },
  "autor": {
    "id": 1,
    "nombre": "Marta Ruiz Calvo",
    "email": "marta.ruiz@corriente.es",
    "rol": "Editora jefa y diseño",
    "bio": "Lleva quince años maquetando revistas que nadie imprime ya. Dirige Corriente desde su primer número y sigue defendiendo que una retícula bien elegida ahorra más discusiones que cualquier reunión.",
    "avatar": "https://i.pravatar.cc/160?img=47",
    "redes": {
      "youtube": "https://youtube.com/@corriente",
      "facebook": "https://facebook.com/corriente.medio",
      "instagram": "https://instagram.com/marta.disena",
      "x": "https://x.com/marta_disena",
      "linkedin": "https://linkedin.com/in/martaruizcalvo"
    },
    "created_at": "2026-08-01 10:00:00",
    "updated_at": "2026-08-01 10:00:00"
  },
  "categoria": {
    "id": 2,
    "nombre": "Tendencias",
    "slug": "tendencias",
    "created_at": "2026-08-01 10:00:00",
    "updated_at": "2026-08-01 10:00:00"
  },
  "etiquetas": [
    {
      "id": 1,
      "nombre": "Diseño web",
      "slug": "diseno-web",
      "created_at": "2026-08-01 10:00:00",
      "updated_at": "2026-08-01 10:00:00"
    },
    {
      "id": 9,
      "nombre": "Tendencias",
      "slug": "tendencias",
      "created_at": "2026-08-01 10:00:00",
      "updated_at": "2026-08-01 10:00:00"
    },
    {
      "id": 10,
      "nombre": "Artículos",
      "slug": "articulos",
      "created_at": "2026-08-01 10:00:00",
      "updated_at": "2026-08-01 10:00:00"
    }
  ],
  "comentarios": [
    {
      "id": 1,
      "post_id": 1,
      "autor_nombre": "Ana Cortés",
      "autor_email": "ana.cortes@ejemplo.es",
      "cuerpo": "La parte de la retícula visible me ha convencido. Llevamos dos años escondiéndola y el resultado es que nadie sabe por qué las columnas están donde están.",
      "estado": "aprobado",
      "creado_en": "2026-03-15 10:12:00",
      "padre_id": null
    },
    {
      "id": 2,
      "post_id": 1,
      "autor_nombre": "Marta Ruiz Calvo",
      "autor_email": "marta.ruiz@corriente.es",
      "cuerpo": "Gracias, Ana. Justo el mes que viene publicamos un caso con la retícula documentada dentro del propio sitio, a ver qué te parece.",
      "estado": "aprobado",
      "creado_en": "2026-03-15 12:40:00",
      "padre_id": 1
    }
  ],
  "anterior": null,
  "siguiente": {
    "id": 2,
    "titulo": "Quince ejemplos de diseño de una sola página",
    "slug": "quince-ejemplos-de-diseno-de-una-sola-pagina",
    "imagen_portada": "https://picsum.photos/seed/corriente-2/1200/630"
  }
}

"
```

## Prompt 7

**Modelo:** Opus 5.5
**Herramienta:** Claude Code

```
Ah cierto es el estado del comentario no del post! Me gustaría probar que realmente hago un delete y borra uno de los comentarios. Qué comando ejecutar?
```

## Prompt 8

**Modelo:** Opus 5.5
**Herramienta:** Claude Code

```
Perfecto. Lo he comprobado ejecutando "curl -s localhost:3401/posts/diez-tendencias-de-diseno-que-pueden-cambiar-la-web-moderna | jq '.comentarios[] | {id, autor_email}'" para ver los comentarios. He borrado el segundo comentario con "curl -i -X DELETE localhost:3401/comentarios/2 \
  -H 'Content-Type: application/json' \
  -d '{"autor_email":"marta.ruiz@corriente.es"}'" y tras repetir el primer comando he comprobado que se eliminaba. Finalmente lo he restaurado con "make db"
```

## Prompt 9

**Modelo:** Opus 5.5
**Herramienta:** Claude Code

```
C1 tiene tests actualmente?
```

## Prompt 10

**Modelo:** Opus 5.5
**Herramienta:** Claude Code

```
Lo he comprobado y es cierto. Ok entonces actualiza la tabla y di que no tiene tests C1. Anteriormente dijiste "C1 y C2 no dependen de H1: los puse delante porque cortan el daño hoy, y cada uno lleva su propio test." Estoy de acuerdo en que cortan daño, pero al menos C1 no tiene su propio test actualmente
```

## Prompt 11

**Modelo:** Opus 5.5
**Herramienta:** Claude Code

```
He comprobado el codigo de C2 y sí expondría el secreto en Github. Solo vale para validar acciones de moderacion no se firman sesiones. Hay un comentario que admite la mejora "TODO(2024-11): sacarla a variables de entorno antes". Habría que añadir un test e incluso un hook que evite que expongamos secrets en Github
```

## Prompt 12

**Modelo:** Opus 5.5
**Herramienta:** Claude Code

```
Leyendo F2 me parece mucho más grave que exponer una clave en Github para la aplicación. Si algún usuario tiene la clave puede borrar todo y no queda rastro de quien fue y no se puede revocar a usuarios particulares. Este creo que es un buen ejemplo para ilustrar que la urgencia o gravedad no van lo primero
```

## Prompt 13

**Modelo:** Opus 5.5
**Herramienta:** Claude Code

```
He intentado comprobar H9 y no he podido. He lanzado muchos envios en paralelo a ver si alguno recibe comentario que no es el suyo "

for i in $(seq 1 50); do
  curl -s -X POST localhost:3401/posts/diez-tendencias-de-diseno-que-pueden-cambiar-la-web-moderna/comentarios \
    -H 'Content-Type: application/json' \
    -d "{\"autor_nombre\":\"n$i\",\"autor_email\":\"n$i@ejemplo.es\",\"cuerpo\":\"comentario $i\"}" \
  | jq -r --arg yo "n$i" 'select(.comentario.autor_nombre != $yo) | "pedí \($yo), recibí \(.comentario.autor_nombre) <\(.comentario.autor_email)>"' &
done; wait

" . Los cometarios de prueba están en la db uno por envío y los 50 recibieron su propio comentario, así que H9 no queda ni confirmando ni descartado. Ningún envío cayó en el hueco pues jq habría impreso  «recibí null» y no lo hizo. El hueco está en el código (routes.ts:324 → :329-331) pero quizá el hueco sea estrecho para 50 envíos  o el driver de SQLite ejecuta las consultas de forma que en este entorno casi no sucede. Finalmente restauré la db con "make db". Modifica la línea 3 de la parte B: " H9, que con envíos simultáneos uno recibe el comentario y el correo de otro. Lancé 50 POST en paralelo, los 50 recibieron el suyo, y eso no basta para confirmarlo ni para descartarlo"
```

## Prompt 14

**Modelo:** Opus 5.5
**Herramienta:** Claude Code

```
Condensa el comentario de la linea 2 para que sea solo una frase
```

## Prompt 15

**Modelo:** Opus 5.5
**Herramienta:** Claude Code

```
Condensa tamben la tabla porque no cabe en una pantalla. Deberia haber limitado tambien el numero de hallazgos porque en 45 minutos sin tener ni idea del codigo no los puedo comprobar. Limita el número de hallazgos a 10 y/o condensa el contenido de la tabla
```
