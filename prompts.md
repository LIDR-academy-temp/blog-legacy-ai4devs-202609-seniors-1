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

**Modelo:** Opus 5.5 xHigh
**Herramienta:** Claude Code

```
Necesito generar un documento md en esta ubicacion y nombre `docs/auditoria/comentarios.md`.
Ese documento debe contener la auditoria de la funcionalidad comentarios. Solo esa funcionalidad. El resto de funcionalidades (posts, autores,
categorías y etiquetas) quedan excluidas de la auditoria.
Para la auditoria de la funcionalidad comentarios debe cubrir crear, editar, borrar y listar comentarios incluyendo sus rutas en la API, su tabla en la
base de datos y lo que la web enseña de ella. 

El formato no es negociable y debe ser una tabla con este formato

1. **Qué es.** Una frase. Si necesitas tres, es que hay dos hallazgos.
2. **Dónde está**, con **la línea que lo demuestra**. Sin esta casilla, la fila no se escribe.
3. **Qué se rompe, y quién lo nota.** El efecto concreto y a quién le llega. *"Nadie todavía"*
   es una respuesta válida y buena.
4. **Cuántos sitios hay que tocar.** Contados, no estimados.
5. **¿Se deshace con un revert?** Sí o no, y si es que no, por qué no.
6. **¿Bloquea o desbloquea a otro hallazgo?** Nombrando al otro. *"A ninguno"* es la respuesta
   más frecuente.

Debajo de la tabla, en el mismo archivo. Anade los siguientes puntos solo contestando a cuantos hallazgos encontro el agente

1. **Cuántos hallazgos devolvió el agente, y cuántos comprobaste tú abriendo el archivo.** Los
   dos números, tal cual salieron.
2. **El hallazgo del que no supiste decidir la prioridad**, y en una frase, por qué.
3. **Algo que el agente afirmó y no pudiste verificar.** No es lo que dijo mal: es lo que no
   pudiste comprobar ni a favor ni en contra.
```

**Qué salió:** Un subagente auditó en solo lectura y devolvió 32 hallazgos. Lo reanudé dos veces y la segunda vez lo verificó ejecutando: retiró H17, que era falso, y añadió H33. Cada vez reescribió el archivo y se perdieron las correcciones previas. Claude comprobó los 33 abriendo los archivos, relanzó las 16 pruebas de verificación, volvió a aplicar 6 correcciones y rellenó la Parte B.

## Prompt 2

**Modelo:** Opus 5.5 xHigh
**Herramienta:** Claude Code (subagente de auditoría reanudado desde su transcripción)

```
Guarda en el fichero `docs/auditoria/comentarios.md` lo que has encontrado
```

**Qué salió:** Funcionó a la primera. El agente escribió la tabla de 32 hallazgos con una columna extra, «Confianza», y dejó la Parte B pendiente.

## Prompt 3

**Modelo:** Opus 5.5 xHigh
**Herramienta:** Claude Code (subagente de auditoría reanudado desde su transcripción)

```
Utiliza el commando  make test para chequear la suite y realiza la revision otra vez backend/node_modules y frontend/node_modules estan disponibles
```

**Qué salió:** La suite estaba en verde, con 1 test. La verificación en ejecución retiró H17 por falso, bajó H4 a confianza baja y añadió H33. Pero el agente reescribió el archivo entero y borró la Parte B y las correcciones que ya estaban aplicadas.
