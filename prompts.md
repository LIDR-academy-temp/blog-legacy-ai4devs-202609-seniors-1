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
resumí los últimos commits
```

## Prompt 2

**Modelo:** Opus 5.5
**Herramienta:** Claude Code

```
busca en que archivos aparece la palabra tipografia
```

## Prompt 3

**Modelo:** Opus 5.5
**Herramienta:** Claude Code

```
Realicemos la auditoría priorizada de la feature de comentarios y déjala escrita en docs/auditoria/comentarios.md
Tené en cuenta que Una feature es funcionalidad de punta a punta (sus rutas en la API, su tabla en la base de datos y lo que la web enseña de ella, etc). Hay cinco features (posts, comentarios, autores, categorías y etiquetas), pero solo hagamos la auditoría de comentarios: crear, editar, borrar y listar comentarios, en esas tres capas. 

Seguir el siguiente formato: 
- una fila por hallazgo
- seis casillas / columnas:
    1. Qué es: en una frase. necesitas Si tres, es que hay dos hallazgos.
    2. Dónde está: la línea que lo demuestra. Si no se puede demostrar, la fila no se escribe.
    3. Qué se rompe, y quién lo nota. El efecto concreto y a quién le llega. "Nadie todavía" es una respuesta válida y buena.
    4. Cuántos sitios hay que tocar. Contados, no estimados.
    5. ¿Se deshace con un revert? Sí o no, y si es que no, por qué no.
    6. ¿Bloquea o desbloquea a otro hallazgo? Nombrando al otro. "A ninguno" es una respuesta válida.

Establecer el siguiente orden: 
- lo que abarata o desbloquea a otro hallazgo va antes
- dentro de cada nivel, el daño dividido por el radio. 
- Si un hallazgo tiene el daño más alto y la precedencia más baja a la vez, se parte en dos filas, una contención que corta el daño hoy y el arreglo de fondo que espera su turno. 

Las propuestas que nadie ha medido todavía van fuera de la tabla, con lo que habría que medir escrito al lado.

Dejar vacío el resto de las features. No analizarlas.

Guardar cada prompt en el archivo prompts.md
```
