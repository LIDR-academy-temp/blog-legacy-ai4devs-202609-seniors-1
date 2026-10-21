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

## Prompt 

**Modelo:** Opus 5.5 High
**Herramienta:** Claude Code

```
Audita la feature de comentarios. Limítate solo a esta feature.

No modifiques nada. Limitate a contestar. 
Tampoco toques nada del repositorio hasta que te indique.

Audítalo separando los hallazgos en 3 capas: tabla de la base de datos, rutas de la API, y web.



Para cada hallazgo, indica:
	- al menos un archivo y línea de código donde se vea el problema (Si hay varios sitios, indícalos todos). Si no se puede determinar el código concreto, anótalo al final como posible mejora.

	- Que rompe, y a quien afecta.

	- El Radio: En cuantos sitios hay que tocarlo (lista concreta numerada, con ficheros).

	- Si es reversible o no (afecta a datos ya guardados que cambiarán, o que se van a guardar de forma distinta una vez aplicado el cambio) 

	- la precedencia (Si hay otros hallazgos encadenados o relacionados que queden afectados al solucionar el hallazgo actual.)


Ordena los hallagzos por:    
1- la precedencia (lo que abarata o desbloquea a otro hallazgo va antes)
2- dentro de cada nivel, el daño dividido por el radio. (Radio = nº de sitios que hay que tocar para resolverlo).

En los resultados incluye también un valor numérico que hayas usado como precedencia para ordenar. El valor usado como daño, y el radio que hayas calculado.



```

**Qué salió:** Genera cada capa en un formato distinto (Con mucho ruido). Detalla mecanismo de puntuación de los distintos criterios de ordenación

## Prompt 

**Modelo:** Opus 5.5 High
**Herramienta:** Claude Code

```
Genera una tabla completa con todos los hallazgos, incluyendo una columna para identificar la capa (tabla, rutas o web). E mismo formato que has usado para la capa web, pero completa.
```

**Qué salió:** 

## Prompt 

**Modelo:** Opus 5.5 High
**Herramienta:** Claude Code

```
Escribe la tabla resultante, posibles mejoras, y al final los criterios de calculo y de ordenación en el archivo /home/emiralles/LIDR/modulo6/prompts y
  resultado/tablaResultadoClaude.md
```

**Qué salió:** 

## Prompt 

**Modelo:** Opus 5.5 High
**Herramienta:** Claude Code

```
el hallazgo W4, depende de A1. ¿Porque tiene prioridad 1?

```

**Qué salió:** Error de redacción de precedencia. Realmente no hay precedencia. Corrijo yo manualmente.
