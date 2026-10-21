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

**Modelo:** Opus 5.5
**Herramienta:** Claude Code

```
Hazme un resumen de los ultimos commits del repositorio
```

**Qué salió:** Describió los tres commits, solo el primero subió código


## Prompt 2

**Modelo:** Opus 5.5
**Herramienta:** Claude Code

```
Busca en que archivos aparece la palabra "diseño"
```

**Qué salió:** Indico que aparece en 4 archivos (sin diferenciar mayusculas y minusculas) y los enumeró

## Prompt 3

**Modelo:** Opus 5.5
**Herramienta:** Claude Code

```
Busca en que archivos aparece la palabra "diseño"
```

**Qué salió:** Indico que aparece en 4 archivos (sin diferenciar mayusculas y minusculas) y los enumeró

## Prompt 4

**Modelo:** Opus 5.5
**Herramienta:** Claude Code

```
No cambies nada en el codigo del proyecto, ni ejecutes commits, pushs o PRs sin que te lo pida explicitamente
```

**Qué salió:** Indico que todo entendido

## Prompt 5

**Modelo:** Opus 5.5
**Herramienta:** Claude Code

```
Vamos a trabajar especifica y exclusivamente sobre la feature de comentarios. Este blog tiene cinco (posts, comentarios, autores, categorías y etiquetas) y la que nos ocupa es comentarios: crear, editar, borrar y listar comentarios. Una feature es una funcionalidad del blog de punta a punta: sus rutas en la API, su tabla en la base de datos y lo que la web enseña de ella. Trabajaremos unicamente sobre la feature comentarios, ninguna mas y no el sistema entero
```

**Qué salió:** Indico que todo entendido, y que el ejercicio pide registrar cada prompt en prompts.md


## Prompt 6

**Modelo:** Opus 5.5
**Herramienta:** Claude Code

```
No edites tampoco el fichero prompts.md. Yo me ocupo
```

**Qué salió:** Indico que entendido

## Prompt 7

**Modelo:** Opus 5.5
**Herramienta:** Claude Code

```
Realiza una auditoría priorizada de la feature de comentarios y déjala escrita en docs/auditoria/comentarios.md

Y el orden de esta prioridad debe ser el siguiente: primero la precedencia (lo que abarata o desbloquea a otro hallazgo va antes), y dentro de cada nivel, el daño del hallazgo dividido por el radio (cuantos sitios hay que tocar para corregirlo). Si un hallazgo tiene el daño más alto y la precedencia más baja a la vez, dividelo en dos filas: una con una contención que corta el daño hoy y otra con el arreglo de fondo que espera su turno. Y los hallazgos consistentes en propuestas que nadie ha medido todavía van fuera de la tabla, indicado al lado que habría que medir para que se pueda incluir de nuevo en la tabla

El formato no es negociable: una fila por hallazgo, con estas seis casillas:

- Qué es. Una frase. Si necesitas tres, es que hay dos hallazgos
- Dónde está, con la línea que lo demuestra. Sin esta casilla, la fila no se escribe
- Qué se rompe, y quién lo nota. El efecto concreto y a quién le llega. "Nadie todavía" es una respuesta válida y buena
- Cuántos sitios hay que tocar. Contados, no estimados
- ¿Se deshace con un revert? Sí o no, y si es que no, por qué no
- ¿Bloquea o desbloquea a otro hallazgo? Nombrando al otro. "A ninguno" es la respuesta más frecuente
```

**Qué salió:** Indico que entendido


