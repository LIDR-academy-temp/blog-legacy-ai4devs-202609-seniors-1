# Prompts

**Modelo:** Claude Opus 5 · **Herramienta:** Claude Code (app de escritorio) en todos.

> Trabajo en una conversación continua, así que los prompts son cortos: el contexto ya está puesto
> de las lecciones. Los pego tal cual, incluidos los que no van del ejercicio.

---

## Prompt 1

```
vamos por la lectura del modulo 6
```

**Qué salió:** el agente fue **primero a la lección del ejercicio**, no a la teoría, y sacó lo que
importaba: **el plazo**, y que este módulo **cambia de proyecto** —Blog Legacy, no FlowSync—.

---

## Prompt 2 — el que corrigió al agente

```
el chrome esta abierto y hay que marcarlo como completado
```

**Qué salió:** en el módulo 5 el agente leyó las cuatro lecciones **sin marcarlas en la
plataforma**, y tuve que decírselo. Aquí se lo dije **antes de empezar**, y las marcó una a una
según las leía.

> 🔑 **Es la misma corrección dos módulos seguidos.** La primera vez fue un hallazgo; la segunda es
> un patrón, **y por eso dejó de ser un prompt y pasó a ser una regla escrita** en sus notas. Es
> literalmente lo que dice la lección 4: *«la misma advertencia en tres encargos seguidos no es un
> prompt, es una regla del proyecto»*.

---

## Prompt 3

```
solo el ejercicio del 6
```

**Qué salió:** tres palabras, y el agente hizo el entorno entero y la auditoría. **No escribí ni un
prompt sobre comentarios, ni sobre los cuatro ejes, ni sobre el formato de la tabla:** todo eso
estaba en el enunciado y en las notas de las cuatro lecciones, que ya habíamos leído.

---

## Lo que delató el archivo esta vez

### El entorno costó diez veces más que la auditoría

**La auditoría se hizo en 3 minutos de los 45.** El entorno costó cerca de una hora, y por un
motivo que no estaba en ninguna guía:

- `make setup` murió compilando **`better-sqlite3`**: `node-gyp` no encontraba Python.
- La guía del curso dice que en Windows hay que usar **WSL**. **No hizo falta.**
- El arreglo fue apuntar `node-gyp` al **Python que ya venía con Visual Studio**
  (`npm_config_python=".../Shared/Python39_64/python.exe"`). Las herramientas de C++ ya estaban:
  hay VS 2019 y VS 2022 instalados.
- Después faltaba `APP_KEY`, porque `make setup` **murió antes de llegar a generarla**. Un
  `node ace generate:key` y listo.

> 🎯 **Tercera vez que la indicación de «en Windows usa WSL» resulta evitable** en este máster. No
> es que la guía esté mal: es que **da por imposible lo que solo está sin configurar**.

### Y la auditoría fue rápida porque medí antes de escribir

No pedí «audita los comentarios». **Primero levanté la API y consulté la base**, y solo después
escribí filas. Por eso los dos hallazgos de arriba traen números en vez de adjetivos: *«3 correos
en pantalla»* y *«de 9 comentarios, 3 aprobados invisibles»*.

**Lo caro no fue encontrar ni ordenar: fue dejar el proyecto en pie para poder comprobar.** Que es,
con otra cara, lo mismo que decía la lección 2 — **el cuello está en comprobar**.
