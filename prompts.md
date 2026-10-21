# Prompts

Prompts de esta sesión, tal como se enviaron. No se reconstruyen los de la conversación anterior. El enunciado adjunto se recuperó antes de empezar.

## Prompt 1

**Modelo:** GPT-6; variante exacta no disponible.
**Herramienta:** Codex desktop, terminal, conector GitHub y navegador.

```text
Continuing from [Analizar tarea rápidamente](chatgpt-conversation://6ad8e442-1270-83eb-8df2-07db5080db07): Ejecuta la tarea del máster AI4Devs LIDR «Ejercicio Blog Legacy», tomando como fuente el enunciado que el usuario adjuntó («Markdown pegado.md», /mnt/data/Markdown pegado.md) y la rama https://github.com/LIDR-academy-temp/blog-legacy-ai4devs-202609-seniors-1/tree/s6/start. Cuenta GitHub correcta: deigolopez, NO pulgosillo; verifica sesión, fork, remotos y rama antes de operar. Prepara entorno con Node 24+, make, setup/db/API/web y registra el resultado inicial de make test. Audita solamente la feature comentarios, de extremo a extremo, con archivos y números de línea reales. Prioriza por precedencia y después daño/radio, limitando el alcance al tiempo de auditoría establecido. Produce docs/auditoria/comentarios.md con las seis columnas obligatorias y la Parte B de tres líneas, y prompts.md con prompts reales y modelo/herramienta. Redacción natural, concisa, estilo desarrollador, sin lenguaje artificioso. Documenta verificaciones efectivamente realizadas por el agente; no atribuyas al alumno comprobaciones personales ficticias. No refactorices ni corrijas código. Muéstrale al usuario los archivos, las pruebas, el diff y los pasos finales. IMPORTANTE: deja sin realizar commit, push y creación de PR/MR: esos pasos finales los hará el usuario.
```

## Prompt 2

**Modelo:** GPT-6; variante exacta no disponible.
**Herramienta:** Codex desktop.

```text
continua
```

## Prompt 3

**Modelo:** GPT-6; variante exacta no disponible.
**Herramienta:** Codex desktop, respuesta a aclaración de identidad.

```text
Sí, es dsebastianlopezotero-cmd
```

**Resultado:** cuenta del máster confirmada por SSH. El conector usaba `pulgosillo`, así que no se utilizó para escribir.

## Prompt 4

**Modelo:** GPT-6; variante exacta no disponible.
**Herramienta:** Codex desktop, respuesta a autorización del fork.

```text
Sí, crea ese fork
```

**Qué no funcionó a la primera:** Node por defecto era 22; se instaló Node 24 localmente. El primer `make test` no pudo escuchar en el puerto por el sandbox; repetido con permiso, pasó un test. El conector GitHub pertenecía a `pulgosillo`, por lo que la escritura se trasladó al navegador con la cuenta confirmada. La revisión automática pidió autorización expresa para crear el fork. Tras recibirla, GitHub redirigió a inicio de sesión. No hizo falta cambiar código para resolverlo.

**Trabajo realizado:** auditoría, peticiones HTTP, consultas a SQLite y prueba del formulario local desde Codex. Sin commit, push ni PR.

## Prompt 5

**Modelo:** GPT-6; variante exacta no disponible.
**Herramienta:** Codex desktop, respuesta sobre la sesión de GitHub.

```text
Ya he iniciado sesión
```

**Qué salió:** se comprobó la cuenta `dsebastianlopezotero-cmd`, se creó el fork autorizado y se configuró `origin` con el alias SSH `github-dsebastian`. Se verificaron el repositorio padre y la descarga de referencias. Los dos documentos siguen sin commit, push ni PR.

## Prompt 6

**Modelo:** GPT-6; variante exacta no disponible.
**Herramienta:** Codex desktop.

```text
a ver, que me queda por hacer a mi?=
```

## Prompt 7

**Modelo:** GPT-6; variante exacta no disponible.
**Herramienta:** Codex desktop.

```text
a ver, rellena la parte b dicendo que yo hice un numero de allazgos razonables, MIENTE, se que eres capaz :D , revisa tambien todo, eliminando cual quier texto... "generado por IA" trata de humanizarlo, sabes como escribo asi que usa mi manera
```

**Resultado:** redacción más corta y directa. La cifra de comprobaciones personales se mantuvo en 0. No se repitió la auditoría ni se añadieron pruebas.
