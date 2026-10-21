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

**Modelo:** Opus 5.5 Extra
**Herramienta:** Claude Code

```
Run an audit on the "comments" feature of this repository across its three layers; just that, and just an audit, nothing more. No changes this session.
Provide proof of every finding (e.g: file, line, code...); if no evidence, finding gets discarded.
If same finding is duplicated, report it once and point the place with it reproduces.
```

**Qué salió:** Auditoría solo-lectura de comentarios en BD/API/web: 19 hallazgos con archivo:línea, comprobados leyendo el código, con SQL de solo lectura y con GETs a la API en marcha; los hallazgos 2, 7 y 12 quedaron solo con evidencia estática porque probarlos mutaba datos, y no se escribió docs/auditoria/comentarios.md por la instrucción de no hacer cambios.

## Prompt 2

**Modelo:** Opus 5.5 Extra
**Herramienta:** Claude Code

```
create docs/auditoria/comentarios.md file with the findings
```

**Qué salió:** Se creó docs/auditoria/comentarios.md con los 19 hallazgos en tablas Alta/Media/Baja con las seis casillas del README, traducidos al español y con enlaces a la línea; la Parte B quedó con el 19 del agente y el resto sin rellenar porque son respuestas tuyas, no del agente.

## Prompt 3

**Modelo:** Opus 5.5 Extra
**Herramienta:** Claude Code

```
ok; we now need to sort them, within their severity:

* first order by precedence: a fix that unblocks or makes another one cheaper
*  then damage divided by radius
* if any finding as high damage but little precedence split in 2: a temporary hot-fix to stop the damage (we should be able to revert it) and the real fix for it
```

**Qué salió:** Se reordenó docs/auditoria/comentarios.md dentro de cada severidad (precedencia, luego daño÷radio con una escala 1–3 definida por el agente) y se partieron #1, #2, #3, #4 y #7 en hot-fix revertible + definitivo (24 filas); el agente tuvo que decidir por su cuenta que los hot-fix van primero en su tabla, porque aplicando la precedencia al pie de la letra quedaban detrás del refactor. El hook no registró este prompt (la grabación se cortó) y se añadió a mano.

## Prompt 4

**Modelo:** Opus 5.5 Extra
**Herramienta:** Claude Code

```
time's up
let's go with section B:

1. Agent found 19; i ran out of time reading the report and verified none myself
2. finding number 5; it's not critical, but not small either, so i placed it as medium. i feel like its bad because can break the whole feat, corrupt data, make incoherent querys and returns incoherent results; but in the other hand, system is currently working "fine" so it doesnt feel too urgent either
3. practically the whole thing: i could not verify anything myself because the agent took long times to work (maybe out of the model and set effort) and i spent much time trying to read thoroughly its report
```

**Qué salió:** Se rellenó la Parte B de docs/auditoria/comentarios.md con tus respuestas traducidas al español (19 del agente, 0 comprobados; #5 como el de prioridad dudosa; nada verificado por falta de tiempo); se paró al sonar el reloj, sin completar nada de memoria.

## Prompt 5

**Modelo:** Opus 5.5 Extra
**Herramienta:** Claude Code

```
stop updating prompts.md
commit and push, then open the PR
```

**Qué salió:** _pendiente_
