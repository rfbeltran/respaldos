# Respaldos

Copias de seguridad de los demás repositorios, en formato **`git bundle`**: un
archivo por respaldo, con toda la historia y todas las refs del repo de origen.

Existe para que los respaldos vivan en un **sitio físicamente distinto** de la
máquina donde se trabaja. La regla que lo obliga está en `CLAUDE.md.global` §7,
"Si el cambio exige respaldo, el respaldo va por triplicado".

---

## Qué hay aquí, y qué no

| | Dónde vive | Por qué |
|---|---|---|
| **Bundles** (`.bundle`) | Aquí, una carpeta por repo | Solo contienen lo versionado, así que pueden salir de la máquina |
| **Tar del repo** (`.tar.gz`) | Solo en `~/respaldos-claude/`, **nunca aquí** | Incluyen los archivos que `.gitignore` excluye: accesos y configuración local |

> ⚠️ **Los tar no se suben.** Su valor está justo en llevar lo que git ignora, que
> son datos privados. Subirlos a un repositorio —aunque sea este y aunque sea
> privado— es exactamente la fuga que `CLAUDE.md.global` §7 prohíbe.
>
> **Decidido el 2026-08-16: los archivos locales mueren con la máquina.** No se
> protegen ni se replican; en un equipo nuevo se vuelven a crear desde cero. El
> tar sirve para deshacer un error del día, no para sobrevivir a un disco muerto.

---

## Estructura

Una carpeta por repositorio de origen, con el nombre del repo:

```
respaldos/
  README.md
  <nombre-del-repo>/
    <nombre-del-repo>-AAAAMMDD-HHMMSS.bundle
```

La fecha y hora del nombre son las de creación del bundle, y son lo que permite
saber a qué estado corresponde sin abrirlo.

---

## Cómo crear un respaldo

Desde el repositorio que se quiere respaldar:

```bash
REPO=$(basename "$PWD")
DEST=~/proyectos/respaldos/$REPO
mkdir -p "$DEST"
B="$DEST/$REPO-$(date +%Y%m%d-%H%M%S).bundle"

git bundle create "$B" --all
git bundle verify "$B"        # debe decir: records a complete history
```

Y después, en este repo:

```bash
cd ~/proyectos/respaldos
git add . && git commit -m "Respaldo de <repo>: <motivo>" && git push
```

**Verificar siempre.** Que `git bundle create` no dé error no significa que el
archivo sirva; `verify` es lo que confirma que la historia está completa, y es
justo lo que un `tar` de `.git` no sabe decirte.

---

## Cómo restaurar

Un bundle se comporta como un remoto: se clona igual que una URL.

```bash
# recuperar el repo entero en un directorio nuevo
git clone <archivo>.bundle repo-recuperado

# o traer una sola rama a un repo que ya existe
git fetch <archivo>.bundle main

# ver qué contiene, sin extraer nada
git bundle list-heads <archivo>.bundle
git bundle verify     <archivo>.bundle
```

Lo que se recupera es **todo lo versionado**: historia completa, todas las ramas
y las refs remotas del momento del respaldo. Lo que **no** se recupera son los
archivos ignorados por git, por lo dicho arriba.

---

## El procedimiento está ensayado de punta a punta

No es teoría: el 2026-08-16 se creó el primer respaldo de `Mejoras_Claude-Cli` y
**se restauró desde el bundle** para comprobar que el camino de vuelta funciona.

| Comprobación | Resultado |
|---|---|
| `git bundle verify` | *The bundle records a complete history* |
| Clonado del bundle a un directorio limpio | ✅ sin errores |
| Commits recuperados | **42**, los mismos que el original |
| SHA del último commit | **idéntico** al original (`c90a4af…`) |
| Archivos versionados | 12, los mismos |
| Archivos ignorados por git recuperados | **0** — como debe ser |
| Tamaño del bundle | 108 KB |
| Tamaño del tar equivalente | 416 KB, 262 archivos |

Dos cosas que este ensayo deja medidas, no supuestas:

**El bundle no lleva nada ignorado.** Cero archivos `.local` en lo restaurado.
Por eso puede salir de la máquina sin violar el §7, y por eso el tar no puede.

**El tar pesa cuatro veces más que el bundle** porque lleva cosas distintas: el
`.git` empaquetado y el árbol de trabajo completo, incluidos los archivos que
git ignora. No se sustituyen entre sí; cada uno cubre un fallo.

> ⚠️ **`verify` no basta.** Comprueba que el archivo está íntegro, no que sepas
> restaurarlo. La única comprobación que no puede dar un falso positivo es
> clonar el bundle y contar lo que sale. Al crear un respaldo importante,
> hazlo.

## Pendientes

Formato del listado —casillas, numeración que no se recicla, fechas y
comentarios que se añaden en vez de sustituir— en `CLAUDE.md.global` §5. Contar
es directo: `grep -c '^- \[ \]' README.md`.

*Ninguno abierto por ahora.*

---

## Por qué un repo aparte y no un bundle dentro de cada repo

Porque el tamaño se dispararía. Un bundle contiene la historia entera, incluidos
los bundles commiteados antes: un repo de 1 MB pasaría a 2 MB al guardar su
primer bundle, y el segundo bundle pesaría ya esos 2 MB. Cada respaldo duplicaría
al anterior. Además los binarios no se comprimen bien entre versiones y no se
pueden quitar sin reescribir la historia — que es precisamente de lo que el
bundle protege.
