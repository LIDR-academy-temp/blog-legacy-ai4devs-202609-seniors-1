# Prompts

---

## Prompt 1

**Modelo:** Opus 5.5 **Herramienta:** Claude Code **Tiempo**: 5m

```
Vamos a realizar una auditoria de la feature comentarios única y exclusivamente, obviando el resto de features.
Esto es una auditoria por lo tanto **NO DEBES** realizar modificaciones, borrado sobre ficheros o base de datos. Solamente 
consultas y análisis.
Audita la feature por completo, desde las rutas de la API, sus registros en tabla, lo que enseña en la web y lista los hallazgos.
Por definición, un hallazgo es una afirmación concreta que alguien puede ir a comprobar.
Cada hallazgo debe ser posible comprobarlo, si no se puede comprobar no es un hallazgo es una suposición.
Por cada hallazgo crea una tabla con 6 columnas que te paso a detallar:
- Que es? En un sola frase, si hay más de una son varios hallazgos
- Donde está? La linea que lo demuestra. Si no está, la linea no se escribe porque entra en contradicción con la definición de hallazgo
- Que se rompe, y quién lo nota. A quien le llega, si no llega a nadie o nadie lo nota, la respuesta "Nadie todavía" es correcta.
- Cuantos sitios hay que tocar. Contados reales, sin estimaciones
- ¿Se deshace con un revert? Si o no, en caso de ser un no, explica el motivo resumido
- ¿Bloquea o desbloquea otro hallazgo? Nombra a cual, en caso de negativo "A ninguno" es una respuesta correcta.
Si encuentras cualquier otro hallazgo que no se contemple en esta tabla, dejalo anotado al pie de la misma.
```

**Qué salió:** (opcional, una línea)  Mostró la tabla y datos complementarios a la auditoria requerida

## Prompt 2

**Auditoria adversarial con otro agente** **Modelo:** GPT 6.1 Sol **Herramienta:** Codex **Tiempo**: 3m  
Se realiza una auditoria extra fuera de los requisitos del ejercicio a modo de comparar ambos resultados
con otro agente/modelo usando el mismo prompt.

```markdown

Vamos a realizar una auditoria de la feature comentarios única y exclusivamente, obviando el resto de features.
Esto es una auditoria por lo tanto NO DEBES realizar modificaciones, borrado sobre ficheros o base de datos. Solamente
consultas y análisis.
Audita la feature por completo, desde las rutas de la API, sus registros en tabla, lo que enseña en la web y lista los hallazgos.
Por definición, un hallazgo es una afirmación concreta que alguien puede ir a comprobar.
Cada hallazgo debe ser posible comprobarlo, si no se puede comprobar no es un hallazgo es una suposición.
Por cada hallazgo crea una tabla con 6 columnas que te paso a detallar:
- Que es? En un sola frase, si hay más de una son varios hallazgos
- Donde está? La linea que lo demuestra. Si no está, la linea no se escribe porque entra en contradicción con la definición de hallazgo
- Que se rompe, y quién lo nota. A quien le llega, si no llega a nadie o nadie lo nota, la respuesta "Nadie todavía" es correcta.
- Cuantos sitios hay que tocar. Contados reales, sin estimaciones
- ¿Se deshace con un revert? Si o no, en caso de ser un no, explica el motivo resumido
- ¿Bloquea o desbloquea otro hallazgo? Nombra a cual, en caso de negativo "A ninguno" es una respuesta correcta.
  Si encuentras cualquier otro hallazgo que no se contemple en esta tabla, dejalo anotado al pie de la misma.

Esta auditoria comparala con tu propio criterio contra la actual, llamada comentarios.md dentro de la carpeta \wsl.localhost\Ubuntu\home\tony\repos\blog-legacy-ai4devs-202609-seniors-1\auditoria\comentarios.md y verifica si coincide o no con tus hallazgos.
```

**Qué salió:** (opcional, una línea)  Mostró la tabla y datos complementarios comparándola con la auditoria previa indicando que encontró 4 que no fueron detectados con Opus y reflejandolo en el documento como PATCH



## Prompt 3

**Reordenación de la tabla** **Modelo:** Opus 5.5 **Herramienta:** Claude **Tiempo**: 1m 30s

```markdown
Reordena la tabla de hallazgos de `docs/auditoria/comentarios.md` con el criterio del mentor.
No toques las demás secciones del archivo (revisión adversarial, «Otros hallazgos», Parte B).
No modifiques código ni la BD; solo puedes hacer consultas de lectura (por ejemplo, `git log -S`).

## Criterio
1. **Primero la precedencia:** lo que abarata o desbloquea otro hallazgo va antes. Niveles:
   - `C`: contención que corta el daño hoy.
   - `N0`: precede a todo. Por ejemplo, los tests de caracterización, porque CLAUDE.md obliga a
     escribirlos antes de corregir.
   - `N1`: abarata o desbloquea otro hallazgo. Distingue "bloquea" (dependencia real) de
     "abarata" (sin él se puede, pero cuesta más).
   - `N2`: no desbloquea ninguno.
2. **Dentro de cada nivel, daño ÷ radio**, de mayor a menor.
   - Daño de 1 a 5, con la escala explicada encima de la tabla: 5 = pérdida irreversible o datos
     personales expuestos hoy; 1 = latente, nadie lo nota.
   - Radio = columna «Sitios a tocar», con recuentos reales. Si al revisar un recuento resulta
     incorrecto, corrígelo y dilo.
   - Cada fila empieza con `[nivel · daño/radio]`.
3. **Si un hallazgo tiene el daño más alto y la precedencia más baja, pártelo en dos filas:**
   - `Hna`: la contención que corta el daño hoy. Comprueba quién la notaría; por ejemplo, busca en
     el frontend si alguien usa el endpoint que vas a retirar.
   - `Hnb`: el arreglo de fondo, que espera su turno.
4. **Las propuestas que nadie ha medido van fuera de la tabla,** en una sección «Propuestas sin
   medir», cada una con lo que habría que medir al lado (logs de acceso, prueba concurrente en la
   BD de test, `EXPLAIN QUERY PLAN`, volumen de spam…).

## Se mantiene
- Las 6 columnas de la tabla y el texto de cada hallazgo, salvo correcciones comprobadas.

```

**Qué salió:** (opcional, una línea)  Mostró la tabla ordenada según los criterios mostrados.

