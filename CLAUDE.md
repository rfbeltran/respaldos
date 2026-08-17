# respaldos — índice

Copias de seguridad de los demás repositorios en formato **`git bundle`**: un
archivo por respaldo, con toda la historia y todas las refs del repo de origen.
Existe para que los respaldos vivan en un **sitio físicamente distinto** de la
máquina donde se trabaja.

La versión completa —estructura, comandos y el porqué de cada decisión— está en
[`README.md`](README.md). Aquí va lo que hay que saber sin abrirlo.

La regla que obliga a todo esto es `CLAUDE.md.global` §7, *"Si el cambio exige
respaldo, el respaldo va por triplicado"*. El global vive versionado en el repo
`Mejoras_Claude-Cli`.

---

## Reglas duras

1. **Aquí solo entran bundles.** Nada de `.tar.gz`. El `.gitignore` los bloquea,
   pero la regla es anterior al `.gitignore`: un tar de un repo lleva los
   archivos que su `.gitignore` excluye —accesos, configuración local— y subirlos
   es la fuga que el §7 prohíbe, **también en un repo privado**.
2. **Los tar se quedan en `~/respaldos-claude/`** de la máquina, y no salen de
   ahí. Decidido el 2026-08-16: los archivos locales mueren con la máquina y se
   rehacen desde cero en una nueva.
3. **Un bundle sin `verify` no es un respaldo.** Que `git bundle create` no dé
   error no significa que el archivo sirva.
4. **Nunca se guarda el bundle de un repo dentro de ese mismo repo**: cada bundle
   contendría a los anteriores y el tamaño se duplicaría en cada copia. Por eso
   existe este repositorio.
5. **Una carpeta por repo de origen**, y el nombre del archivo lleva fecha y hora:
   `<nombre-del-repo>/<nombre-del-repo>-AAAAMMDD-HHMMSS.bundle`.

## Crear un respaldo

Desde el repo que se quiere respaldar:

```bash
REPO=$(basename "$PWD"); DEST=~/proyectos/respaldos/$REPO; mkdir -p "$DEST"
B="$DEST/$REPO-$(date +%Y%m%d-%H%M%S).bundle"
git bundle create "$B" --all
git bundle verify "$B"        # debe decir: records a complete history
cd ~/proyectos/respaldos && git add . && git commit && git push
```

## Restaurar

```bash
git clone <archivo>.bundle repo-recuperado   # el repo entero
git fetch <archivo>.bundle main              # solo una rama, a un repo existente
git bundle list-heads <archivo>.bundle       # ver qué trae, sin extraer
```

Se recupera **todo lo versionado**: historia, ramas y refs remotas del momento
del respaldo. **No** se recuperan los archivos ignorados por git.

## Acceso

Clon: `git clone git@github-respaldos:rfbeltran/respaldos.git` — alias de
`~/.ssh/config` con llave dedicada, igual que los demás repos.

---

## Pendientes

Formato del listado —casillas, numeración que no se recicla, fechas, comentarios
que se añaden— en `CLAUDE.md.global` §5. La lista completa, con el historial de
comentarios, va en [`README.md`](README.md).

*Ninguno abierto por ahora.*

---

*Índice verificado contra el estado del repo el **2026-08-16**.*
