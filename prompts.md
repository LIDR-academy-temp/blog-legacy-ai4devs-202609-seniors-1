# Prompts

## Prompt 1
Generame la auditoría priorizada de la feature de comentario
y déjala escrita en docs/auditoria/comentarios.md
Analiza sus rutas en la API, su tabla en la base de datos y lo que la web enseña de ella

sobre el formato muestrame una fila por hallazgo, con estas seis casillas.

Qué es. Una frase. Si necesitas tres, es que hay dos hallazgos.

Dónde está, con la línea que lo demuestra. Sin esta casilla, la fila no se escribe.

Qué se rompe, y quién lo nota. El efecto concreto y a quién le llega. "Nadie todavía" es una respuesta válida y buena.

Cuántos sitios hay que tocar. Contados, no estimados.

¿Se deshace con un revert? Sí o no, y si es que no, por qué no.

¿Bloquea o desbloquea a otro hallazgo? Nombrando al otro. "A ninguno" es la respuesta más frecuente.
