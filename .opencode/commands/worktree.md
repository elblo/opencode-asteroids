---
description: Crea un worktree local a partir del argumento y su contexto
---

Interpreta el argumento completo, que puede contener espacios, como contexto para elegir un nombre breve y descriptivo para el worktree. Si no se proporciona argumento, usa el contexto de la conversación; si no hay contexto suficiente para elegir un nombre, pide uno y no ejecutes nada.

Normaliza el nombre a un único segmento seguro para ruta y rama: minúsculas, sin acentos, palabras separadas por guiones y solo caracteres `a-z`, `0-9` y `-`. No incluyas `.worktrees/` en el nombre.

Después ejecuta únicamente este comando, sustituyendo `<nombre-del-worktree>` por el nombre elegido:

```sh
git worktree add .worktrees/<nombre-del-worktree>
```

No cambies de directorio, no crees carpetas ni ejecutes comandos adicionales. Informa brevemente del resultado del comando.
