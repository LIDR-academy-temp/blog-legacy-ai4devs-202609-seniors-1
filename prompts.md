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

**Modelo:** Opus 5.5 Medium
**Herramienta:** Claude Code

```
resumeme los ultimos commits
```

**Qué salió:** resumió los 3 commits de `s6/start` (import inicial, README con requisitos y enlaces de la cohorte) y avisó de cambios sin commitear en `.claude/settings.json`, `prompts.md` y tres ficheros nuevos. (1 herramienta)

## Prompt 2

**Modelo:** Opus 5.5 Medium
**Herramienta:** Claude Code

```
busca la palabra "autor" en el repo y dime en que archivo y linea se encuentra cada hallazgo
```

**Qué salió:** buscó «autor» como palabra completa en los ficheros versionados y listó las 50 coincidencias con fichero y línea, agrupadas en documentación, backend y frontend; ofreció las 154 que incluyen variantes. (1 herramienta)

## Prompt 3

**Modelo:** Opus 5.5 Medium
**Herramienta:** Claude Code

```
lee el README del repo para que tengas el contexto del ejercicio a realizar en esta sesion. Solo lee, no empieces la tarea ni me hagas comentarios. Puedes hacerme preguntas en caso de que detectes huecos o te falte contenido para entender el ejercicio
```

**Qué salió:** leyó el README del ejercicio sin empezar la tarea y preguntó cuatro cosas antes de seguir: nombre de rama (`auditoria-<iniciales>` frente a `refactor/` o `feat/`), si `prompts.md` cuenta como parte del ejercicio, si el reloj de 45 minutos y (1 herramienta)

## Prompt 4

**Modelo:** Opus 5.5 Medium
**Herramienta:** Claude Code

```
1. la rama sera auditoria-epo y no se hace hasta el final, ya que solo entregaremos lo que piden, no mis hooks para escribir prompts.md u otros archivos de apoyo al ejercicio.
2. prompts.md esta bien tal cual, los prompts que aparecen hasta ahora son parte del ejercicio
3. No hace falta, lo hago yo manualmente
4. Lo abres tu, pero antes de dare la orden para que crees la rama de entrega, auditoria-epo, y copiaras en ella solo los archivos que se piden para la entrega. Finalmente crearas la pr con gh pr create. Pero solo cuando yo te lo diga
```

**Qué salió:** un alumno pidió fijar las reglas de entrega antes de tocar nada: la rama `auditoria-epo` se crea solo al final y solo con `docs/auditoria/` y `prompts.md`, sin hooks ni `statusline.sh`, y la PR se abre con `gh pr create` solo cuando él lo o (2 herramientas, 1 fichero editado)

## Prompt 5

**Modelo:** Opus 5.5 Medium
**Herramienta:** Claude Code

```
Antes de empezar la auditoria necesito que acotes la funcionalidad "comentarios", que es sobre la que vamos a hacer la auditoria. Rutas api, codigo backend, codigo frontend y tablas de base de datos
```

**Qué salió:** escribe UNA sola línea en español que resuma QUÉ SALIÓ del turno: qué hizo o contestó el asistente, y si funcionó, se quedó a medias o se equivocó. En pasado, sin markdown salvo `backticks` para nombres de fichero, sin comillas alrededor, m (4 herramientas)

## Prompt 6

**Modelo:** Opus 5.5 Medium
**Herramienta:** Claude Code

```
lo incluimos. 
Ahora, necesito que hagas la auditoria centrandote en los hallazgos y sus requisitos:
- Que es
- Donde esta, con archivo y linea que lo demuestre
- Que se rompe con este hallazgo y quien lo nota: usuarios, desarrolladores, pm, etc. O "nadie todavia" si nadie lo esta notando
- Cuantos sitios hay que tocar para arreglar este hallazgo. Tienes que contarlos y no estimarlos, y para ello necesito que me digas donde estan esos sitios
- Este hallazgo, se podria resolver con un arreglo que tuviera un revert limpio? Si o no, y si no lo es, por qué no.
- Este hallazgo, bloquea o desbloquea otro hallazgo? No me contestes solo si o no, dime en caso de que bloquee o desbloquee otro hallazgo, que hallazgo es. Si no bloquea ni desbloquea ninguno, contesta "ninguno"

Tienes que crear una tabla en un archivo ./docs/auditoria/comentarios.md con una fila por hallazgo y una columna por cada casilla de las anteriores, 7 casillas en total. 

No te ofrezcas a auditar el sistema entero, ciñete a la feature "comentarios" que hemos acotado en el encargo anterior.
Hazme las preguntas pretinentes durante la auditoria para poder completarla si te falta informacion
```

**Qué salió:** localizó los hallazgos de comentarios y los confirmó con GET en vivo, corrigió referencias y prioridades, pero se quedó a medias: el turno terminó sin confirmar que `docs/auditoria/comentarios.md` quedara escrito. (8 herramientas, 1 fichero editado)

## Prompt 7

**Modelo:** Opus 5.5 Medium
**Herramienta:** Claude Code

```
perdona, quita la septima casilla, solo son 6.
```

**Qué salió:** quitó la columna «#» para dejar las 6 casillas del README, pasó el id y la prioridad al inicio de «Qué es» y lo explicó encima de la tabla; las tres preguntas abiertas siguen abiertas. (2 herramientas)

## Prompt 8

**Modelo:** Opus 5.5 Medium
**Herramienta:** Claude Code

```
1. Si, "Aprobado" y "ok" cuentan como aprobados
2. No se puede saber, es un hueco de spec. Esto va para la parte B del ejercicio
3. No entiendo como has contado los sitios
```

**Qué salió:** aplicó las decisiones en `docs/auditoria/comentarios.md`: aprobado y ok cuentan como aprobados, y el detalle del post oculta comentarios aprobados (H5); moderación queda sin decidir. (1 herramienta)

## Prompt 9

**Modelo:** Opus 5.5 Medium
**Herramienta:** Claude Code

```
Con respecto a los sitios que hay qie tocar de la casilla 4, no entiendo bien que quiere decir el enunciado: los sitios que hay que tocar con el arreglo, con todos los arreglos, o cuantos archivos y lineas estan relacionados con ese hallazgo
```

**Qué salió:** admitió que el enunciado no define «sitios que tocar» y que su recuento dependía del arreglo elegido; propuso contar dónde está el defecto (recalcula H2, H8 y H10) y preguntó si rehace la columna, sin hacerlo aún. (0 herramientas)

## Prompt 10

**Modelo:** Opus 5.5 Medium
**Herramienta:** Claude Code

```
si, rehaz la columna con ese criterio
```

**Qué salió:** rehízo la columna «Sitios a tocar» contando dónde está cada defecto: H2 baja de 5 a 2, H3 de 3 a 1, H4 de 1 a 0 y H10 sube de 3 a 5; ofreció escribir «¿Se deshace con un revert?» como condición. (1 herramienta)

## Prompt 11

**Modelo:** Opus 5.5 Medium
**Herramienta:** Claude Code

```
Mi problema es que no sé si una auditoria solo le limita a señalar problemas o tb ha de proponer soluciones. El ejercicio no lo aclara
```

**Qué salió:** recomendó que la auditoría solo señale y mida los problemas, sin proponer soluciones, y preguntó si reescribe la casilla 5 de H2, H5, H12 y H15 según el tipo de cambio que exige el arreglo. (0 herramientas)

## Prompt 12

**Modelo:** Opus 5.5 Medium
**Herramienta:** Claude Code

```
sí, reescribe esas celdas de la columna 5
```

**Qué salió:** reescribió las cuatro celdas de revert de la columna 5 (H2, H5, H12 y H15) describiendo qué tipo de cambio exigiría cada arreglo, sin proponer ninguno, y dejó H15 en un «No» sin matices, con la tabla intacta en sus 6 casillas. (2 herramientas)

## Prompt 13

**Modelo:** Opus 5.5 Medium
**Herramienta:** Claude Code

```
revisa la auditoria de neuvo desde esta perspectiva: solo señalar, no proponer soluciones. Las soluciones las diseña el equipo de desarrolladores receptor de la auditoria, que es ademas tb quien ordena los hallazagos segun su criterio. Nosotros nos limitamos a señalar y comprobar los hallazgos para darles una lista curada
```

**Qué salió:** revisó el comentarios.md de la auditoría con criterio de solo señalar: quitó las prioridades y las pistas de solución, añadió una sección sobre qué es y qué no es la auditoría y la forma de comprobar cada hallazgo, y dejó la lectura de la P (3 herramientas, 1 fichero editado)

## Prompt 14

**Modelo:** Opus 5.5 Medium
**Herramienta:** Claude Code

```
ahora vamos hallazgo or hallazgo para que los compruebe manualmente
```

**Qué salió:** se pidió una sola línea que resuma lo que salió del turno; el asistente, sin usar herramientas, dio instrucciones para comprobar el primer hallazgo (H1, el correo del autor de un comentario expuesto en `backend/start/routes.ts`) y dejó el r (0 herramientas)

## Prompt 15

**Modelo:** Opus 5.5 Medium
**Herramienta:** Claude Code

```
solo se ve un comentario, no dos: [Image #1]
```

**Qué salió:** no ejecutó nada: atribuyó a su `grep -A8` el comentario que faltaba, dio un comando curl con una salida esperada sin comprobar y afirmó que la captura muestra el email del autor en la respuesta pública. (0 herramientas)

## Prompt 16

**Modelo:** Opus 5.5 Medium
**Herramienta:** Claude Code

```
comprobado, pasa al H2
```

**Qué salió:** analizó H2: `backend/start/routes.ts` autoriza el `DELETE` solo con el correo del autor, igual que el `PATCH`; dio curls para comprobarlo en vivo y restaurar con `make db`, sin ejecutarlos. (1 herramienta)

## Prompt 17

**Modelo:** Opus 5.5 Medium
**Herramienta:** Claude Code

```


<pasted_content id="8b99">
 blog-legacy-ai4devs-202609-seniors-1 git:(s6/start) ✗ curl -s -i -X DELETE localhost:3401/comentarios/1 -H 'Content-Type: application/json' -d '{"autor_email":"otra@persona.es"}' | head -1
HTTP/1.1 403 Forbidden
➜  blog-legacy-ai4devs-202609-seniors-1 git:(s6/start) ✗ curl -s -X DELETE localhost:3401/comentarios/1 -H 'Content-Type: application/json' -d '{"autor_email":"ANA.cortes@ejemplo.es "}'
{"borrado":true}%
➜  blog-legacy-ai4devs-202609-seniors-1 git:(s6/start) ✗ curl -s localhost:3401/posts/diez-tendencias-de-diseno-que-pueden-cambiar-la-web-moderna | python3 -c "import json,sys; [print(c['id'], c['autor_nombre']) for c in json.load(sys.stdin)['comentarios']]"
2 Marta Ruiz Calvo
➜  blog-legacy-ai4devs-202609-seniors-1 git:(s6/start) ✗ 
</pasted_content id="8b99">
```

**Qué salió:** comprobó H2 con curl (403 con otro correo, borrado con el de Ana en mayúsculas) y marcó `docs/auditor`; avisó de que la respuesta huérfana de Marta sigue en la API, a falta de verificar la web (H12). (1 herramienta, 1 fichero editado)

## Prompt 18

**Modelo:** Opus 5.5 Medium
**Herramienta:** Claude Code

```
comprobado, pasa al H3
```

**Qué salió:** marcó H12 como comprobado en `docs/auditor` y pasó a H3: ubicó la clave hardcodeada en `backend/start/routes.ts`, dio comandos para verificarla y avisó de que el comentario sobre firmar sesiones no se cumple. (1 herramienta, 1 fichero editado)

## Prompt 19

**Modelo:** Opus 5.5 Medium
**Herramienta:** Claude Code

```
[Image #4] [Image #5] [Image #6]  blog-legacy-ai4devs-202609-seniors-1 git:(s6/start) ✗ curl -s -X DELETE localhost:3401/comentarios/1 -H 'x-clave-moderacion: dev-secret-cambiar'
{"borrado":true}%     

la clave se ve en repo publico: [Image #7] he accedido al repo sin logarme en github
```

**Qué salió:** dio H3 por comprobado con un DELETE real: la clave de moderación borra comentarios sin identificarse y se lee en el repo público; actualizó `docs/auditor` y redactó H4 (no hay ruta para aprobar comentarios). (1 herramienta, 1 fichero editado)

## Prompt 20

**Modelo:** Opus 5.5 Medium
**Herramienta:** Claude Code

```
ya se han acabado mis 45 minutos, no puedo seguir comprobando manualmente los hallazgos. Pasemos a la parte B
```

**Qué salió:** el alumno dio por terminados sus 45 minutos sin verificar más hallazgos y pidió pasar a la Parte B; el asistente añadió ese borrador bajo la tabla de `docs/auditoria/comentarios.md` con tres líneas para revisar: 4 hallazgos comprobados (H1, (2 herramientas)

## Prompt 21

**Modelo:** Opus 5.5 Medium
**Herramienta:** Claude Code

```
tengo que anadir que no me ha dado tiempo a establecer la prioridad de los hallazgos
```

**Qué salió:** se añadió al principio de la línea 2 de la Parte B la frase «No me dio tiempo a establecer la prioridad de los hallazgos», pero eso deja una contradicción con la cabecera del archivo (línea 12), que atribuye la falta de prioridad a que la d (2 herramientas, 1 fichero editado)

## Prompt 22

**Modelo:** Opus 5.5 Medium
**Herramienta:** Claude Code

```
la auditoria no lo ordena pero ete ejercicio no es solo hacer la auditoria, sino luego anadir mi criterio como desarrolladora, y la prioridad es criterio que no he podido anadir
```

**Qué salió:** se reescribió la línea 2 de la Parte B de `docs/auditor` para separar que la auditoría no ordena los hallazgos a propósito de que ordenarlos es el criterio del alumno, que no llegó a añadir; H4 quedó como el hallazgo que no supo decidir. (1 herramienta, 1 fichero editado)

## Prompt 23

**Modelo:** Opus 5.5 Medium
**Herramienta:** Claude Code

```
crea la rama auditoria-epo y abre la PR
```

**Qué salió:** creó la rama, commiteó los archivos de entrega y la subió al fork, pero el PR contra `s6/start` quedó a medias: `gh` estaba autenticado con otra cuenta y el reintento con la de elenaPascualO no llegó a confirmarse. (8 herramientas)

## Prompt 24

**Modelo:** Opus 5.5 Medium
**Herramienta:** Claude Code

```
puedes comitear y pushear tb esta rama y subirla a mi fork?
```

**Qué salió:**

## Prompt 25

**Modelo:** Opus 5.5 Medium
**Herramienta:** Claude Code

```
perdon!!
```

**Qué salió:**
