# Capítulo 18: Git

Vim y git son dos grandes herramientas para dos cosas diferentes. Git es una herramienta para el control de versiones. Vim es un editor de texto.

En este capítulo, aprenderás diferentes maneras de integrar Vim y git.

## Destacando las diferencia

Recuerda del capítulo anterior que puedes ejecutar el comando `vimdiff` para mostrar las diferencias entre múltiples archivo.

Supongamos que tienes dos archivos, `archivo1.txt` y `archivo2.txt`. 

Dentro de `archivo1.txt`:

```
tortitas
gofres
manzanas

leche
zumo de manzana

yogur
```

Dentro de `Archivo2.txt`:

```
tortitas
gofres
naranjas

leche
zumo de naranja

yogur
```

Para ver las diferencias entre ambos archivos, ejecuta:

```
vimdiff archivo1.txt archivo2.txt
```

De manera alternativa también podrías ejecutar:

```
vim -d archivo1.txt archivo2.txt
```

<p align="center">
  <img alt="Basic diffing with Vim" width="900" height="auto" src="images/diffing-basic.png">
</p>

`vimdiff` muestra dos *buffers* uno al lado del otro. En la izquierda está `archivo1.txt` y en la derecha está `archivo2.txt`. La primera diferencia (manzanas y naranjas) aparecen resaltadas en ambas líneas.

Supongamos que quieres hacer que el segundo *buffer* tenga también manzanas, en vez de naranjas. PAra transferir el contenido del *buffer* actual (estando el cursor en el archivo `archivo1.txt`) a `archivo2.txt`, primero ve a la siguiente diferencia con `]c` (para saltar a una diferencia previa, utiliza `[c`). El cursor debería estar ahora en manzanas. Ejecuta `:diffput`. Ambos archivos deberían tener ahora la palabra manzanas.

<p align="center">
  <img alt="Diffing Apples" width="900" height="auto" src="images/diffing-apples.png">
</p>

Si necesitas transferir el texto desde el otro *buffer* (zumo de naranja, `archivo2.txt`) para reemplazar el texto del *buffer* actual (zumo de manzana, `archivo1.txt`), con tu cursor todavía en la ventana de `archivo1.txt`, primero dirígete hasta la siguiente diferencia mediante `]c`. Tu cursor ahora debería estar sobre el texto zumo de manzana. Ejecuta `:diffget` para incorporar el texto zumo de naranja desde el otro *buffer* para reemplazar el zumo de manzana en el *buffer* actual.

`:diffput` *pone* el texto desde el *buffer* actual a otro *buffer*. `:diffget` *incorpora* el texto desde otro *buffer* al *buffer* actual.

Si tienes múltiples *buffers*, puedes ejecutar `:diffput archivoN.txt` y `:diffget archivoN.txt` para apuntar al *buffer* del archivoN.

## Vim como herramienta para la fusión (merge)

> "¡Me encanta resolver los conflictos de fusión en git!" - Nadie

No conozco a nadie que le guste resolver conflictos a la hora de fusionar código en git (*merge*). Sin embargo, son inevitables. En esta sección aprenderás cómo aprovechar los recursos de Vim como herramienta para la resolución de conflictos de fusión de código en git.

Primero, cambia la herramienta para realizar la fusión para utilizar `vimdiff` ejecutando:

```
git config merge.tool vimdiff
git config merge.conflictstyle diff3
git config mergetool.prompt false
```

De manera alternativa, puedes modificar directamente en `~/.gitconfig` (de manera predeterminada ese archivo debería estar en la raíz de tu usuario en sistemas basados en Unix, pero tu archivo puede que este en un lugar diferente). Los comandos anteriores deberían modificar tu archivo gitconfig para tener un aspecto similar al texto que se reproduce a continuación, si no todavía no has ejecutado los comandos anteriores, puedes editar manualmente tu archivo gitconfig.

```
[core]
  editor = vim
[merge]
  tool = vimdiff
  conflictstyle = diff3
[difftool]
  prompt = false
```

Vamos a crear un falso conflicto de fusión para probar esto. Vamos a crear un directorio llamado `/comida` y hacer que sea un repositorio de git:

```
git init
```

Añade un fichero llamado `desayuno.txt`. Dentro escribimos:

```
tortitas
gofres
naranjas
```

Añade el archivo y haz un *commit*:

```
git add .
git commit -m "commit inicial de desayuno"
```

Ahora, vamos a crear una nueva rama y llamarla manzanas:

```
git checkout -b manzanas
```

Edita el archivo y cambia el archivo `desayuno.txt`:

```
tortitas
gofres
manzanas
```

Guarda el archivo, añádelo a git y haz un *commit* con los cambios:

```
git add .
git commit -m "Manzanas en vez de naranjas"
```

Genial. Ahora tenemos naranjas en el archivo de la rama principal y manzanas en el archivo de la rama manzanas. Vamos a volver a la rama principal:

```
git checkout master
```

Dentro de `desayuno.txt`, deberías ver naranjas. Vamos a cambiar esa línea a uvas porque son buenas para la salud:

```
tortitas
gofres
uvas
```

Guarda los cambios, añádelo a git y realiza un *commit*:

```
git add .
git commit -m "Uvas en vez de naranjas"
```

Ahora ya puedes realizar una fusión (*merge*) de la rama manzanas a la rama principal:

```
git merge manzanas
```

Deberías ver un error:

```
Auto-merging desayuno.txt
CONFLICT (content): Merge conflict in desayuno.txt
Automatic merge failed; fix conflicts and then commit the result.
```

¡Un conflicto, genial! Vamos a resolver el conflicto utilizando nuestra recién configurada herramienta `mergetool`. Ejecuta:

```
git mergetool
```

<p align="center">
  <img alt="Three-way mergetool with Vim" width="900" height="auto" src="images/mergetool-initial.png">
</p>

Vim muestra cuatro ventanas. Pon atención a las tres superiores:

- `LOCAL` continene `uvas`. Este es el cambio realizado en "local", en el que vas a fusionar.
- `BASE` continene `naranjas`. Este es el ancestro común entre `LOCAL` y `REMOTE` para comparar cómo difieren.
- `REMOTE` continene `manzanas`. Esto es lo que está siendo fusionado.

En la parte inferior (la cuarta ventanas) verás:

```
tortitas
gofres
<<<<<<< HEAD
uvas
||||||| db63958
naranjas
=======
manzanas
>>>>>>> manzanas
```

La cuarta ventana contiene los textos del conflito de fusión. Con este escenario, es sencillo ver qué cambios tiene cada entorno. Puedes ver el contenido de `LOCAL`, `BASE` y `REMOTE` al mismo tiempo. 

Tu cursor debería estar en la cuarta ventana, en el área resaltada. Para obtener los cambios desde `LOCAL` (uvas), ejecuta `:diffget LOCAL`. Para incorporar los cambios desde `BASE` (naranjas), ejecuta `:diffget BASE` y para obtener los cambios desde `REMOTE` (manzanas), ejecuta `:diffget REMOTE`.

En este caso, vamos a incorporar el cambio desde `LOCAL`. Por lo que ejecutaremos `:diffget LOCAL`. En la cuarta ventana ahora tendremos uvas. Guarda y sal de todos los archivos (`:wqall`) cuando hayas terminado. ¿No fue tan mal, verdad?

Quizás has notado que ahora también tienes un archivo llamado `desayuno.txt.orig`. Git crea un archivo de respaldo en caso de que las cosas no hayan salido bien. Si no quieres que git cree ese archivo durante la fusión, ejecuta:

```
git config --global mergetool.keepBackup false
```

## Git dentro de Vim

Vim no tiene una funcionalidad propia para trabajar con git. Una manera de ejecutar comandos de git desde Vim es utilizando el operador `!`, en el modo línea de comandos.

Cualquier comando de git puede ser ejecutado con `!`:

```
:!git status
:!git commit
:!git diff
:!git push origin master
```

También puedes utilizar las convenciones de Vim para el *buffer* actual con `%` o para otro *buffer* con `#`:

```
:!git add %         " git añade el archivo actual
:!git checkout #    " git hace un *checkout* para el archivo alternativo
```

Un truco de Vim que puedes utilizar para añadir múltiples archivos en una ventana diferente de Vim es ejecutar:

```
:windo !git add %
```

Después haz un *commit*:

```
:!git commit "Añadido todo a git en mi ventana de Vim, genial"
```

El comando `windo` es uno de los comandos "do", similar a `argdo` que ya has visto previamente. `windo` ejecuta el comando en cada ventana.

De manera alternativa, también puedes utilizar `bufdo !git add %` para añadir en git todos los *buffers* o `argdo !git add %` para añadir todos los argumentos del archivo, dependiendo de tu forma de trabajar.

## Complementos

Hay muchos complementos para Vim para ayudarte con git. A continuación te muestro una lista de algunos complementos relacionados con git (es probable que existan muchos más cuando estés leyendo esto):

- [vim-gitgutter](https://github.com/airblade/vim-gitgutter)
- [vim-signify](https://github.com/mhinz/vim-signify)
- [vim-fugitive](https://github.com/tpope/vim-fugitive)
- [gv.vim](https://github.com/junegunn/gv.vim)
- [vimagit](https://github.com/jreybert/vimagit)
- [vim-twiggy](https://github.com/sodapopcan/vim-twiggy)
- [rhubarb](https://github.com/tpope/vim-rhubarb)

Uno de los más populares es vim-fugitive. En lo que queda del capítulo, repasaremos cómo es el flujo de trabajo con git utilizando este complemento.

## Vim-fugitive

El complemento vim-fugitive te permite ejecutar los comandos de git para la línea de comandos sin necesidad de abandonar el editor Vim. Verás que algunos comandos son mejores cuando son ejecutados dentro del propio Vim.

Para empezar, vamos a instalar vim-fugitive con un gestor de complementos de Vim ([vim-plug](https://github.com/junegunn/vim-plug), [vundle](https://github.com/VundleVim/Vundle.vim), [dein.vim](https://github.com/Shougo/dein.vim), etc).

## Git Status

Cuando ejecutas el comando `:Git` sin ningún parámetro,vim-fugitive muestra una ventana con un sumario de git. Muestra el archivo o archivos que no tienen seguimiento de git (untracked), que todavía no han sido tomados en cuenta (unstaged) o que ya se han añadido (staged). Mientras estás en este modo "`git status`", puedes realizar varias cosas:

- `Ctrl-N` / `Ctrl-P` ir arriba o abajo en la lista de archivos.
- `-` cambiar el archivo que está bajo el cursor a *stage* o *unstage.
- `s` cambiar a *stage* el archivo que está bajo el cursor.
- `u` cambiar a *unstage* el archivo que está bajo el cursor.
- `>` / `<` para mostrar u ocultar las diferencias mostradas en una sola línea del nombre del archivo bajo el cursor.

<p align="center">
  <img alt="Fugitive Git" width="900" height="auto" src="images/fugitive-git.png">
</p>

Para saber más, echa un vistazo a `:h fugitive-staging-maps`.

## Git Blame

Cuando ejecutas el comando `:Git blame` desde el archivo actual, vim-fugitive muestra una ventana dividida. Esto puede ser útil para encontrar a la persona responsable de escribir esa línea de código que hace que todo falle y poder encontrar al culpable (solo es una broma).

Algunas cosas que puedes hacer mientras estás en el modo `"git blame"`:
- `q` para cerrar la ventana que habre el comando.
- `A` para redimensionar la columna del autor.
- `C` para redimensionar la columna de los *commit*.
- `D` para redimensionar la columna de fecha/hora.

Para más información, echa un vistazo a `:h :Git_blame`.

<p align="center">
  <img alt="Fugitive Git Blame" width="900" height="auto" src="images/fugitive-git-blame.png">
</p>

## Gdiffsplit

Cuando ejecutas el comando `:Gdiffsplit`, vim-fugitive ejecuta un comando `vimdiff` del archivo actual comparándolo con el ínidce del árbol de trabajo. Si ejecutas `:Gdiffsplit <commit>`, vim-fugitive ejecuta `vimdiff` pero esta vez comparándolo con el archivo dentro de `<commit>`.

<p align="center">
  <img alt="Fugitive Gdiffsplit" width="900" height="auto" src="images/fugitive-gdiffsplit.png">
</p>

Debido a que estás en el modo `vimdiff`, puedes *incorporar* o *poner* las diferencias con los comandos `:diffput` y `:diffget`.

## Gwrite y Gread

Cuando ejectuas el comando `:Gwrite` en un archivo después de hacer cambios, vim-fugitive añade los cambios. Es como ejecutar `git add <archivo_actual>`.

Cuando ejecutas el comando `:Gread` en un archivo después de hacer cambios, vim-fugitive restablece el archivo al estado anterior a los cambios. Es como ejecutar `git checkout <archivo_actual>`. Una de las ventajas de ejecutar `:Gread` es que la acción es revertible. Si, después de ejecutar `:Gread`, cambias de opinión y quieres mantener los antiguos cambios, simplemente puedes ejecutar la acción de deshacer (`u`) y Vim revertirá la acción de `:Gread`. Esto no sería posible si hubieras ejecutado `git checkout <archivo_actual>` desde la línea de comandos.

## Gclog

Cuando ejecutas el comando `:Gclog`, vim-fugitive muestra el historial de *commits*. Es similar a ejecutar el comando `git log`. Vim-fugitive utiliza la ventana *quickfix* de Vim para realizar esta tarea, así que puedes utilizar `:cnext` y `:cprevious` para navegar a través de la información ofrecida por el registro de git hacia adelante o atrás. Puedes abrir y cerrar el listado de registros con `:copen` y `:cclose`.

<p align="center">
  <img alt="Fugitive Git Log" width="900" height="auto" src="images/fugitive-git-log.png">
</p>

Mientras estás en el modo `"git log"`, puedes realizar dos cosas:
- Ver el árbol del repositorio git.
- Visitar el *commit* anterior.

De manera similar al comando `git log` también puedes pasarle argumentos a `:Gclog`. Si tu proyecto tiene un historial de *commits* muy extenso y solo necesitar ver los tres últimos *commits*, puedes ejecutar `:Gclog -3`. Si necesitas filtrar por la fecha de *commits*, puedes hacerlo ejecutando algo similar a `:Gclog --after="Enero 1" --before="Marzo 14"`.

## Más Vim-Fugitive

Estos son solo unos pocos ejemplos de lo que puede hacer vim-fugitive. Para aprender más sobre vim-fugitive, echa un vistazo a `:h fugitive.txt`. La mayoría de los comandos más populares de git probablemente están optimizados en vim-fugitive. Solo es necesario que los busques en la documentación.

Si estás dentro de uno de los "modos especiales de vim-fugitive (por ejemplo, dentro de los modos `:Git` o `:Git blame`) y quieres aprender qué atajos de teclado hay disponibles, pulsa `g?`. Vim-fugitive mostrará la ventana `:help` apropiada para el modo en el que estás. ¡Ordenada!

## Aprende Vim y Git de la manera más inteligente

Puede que encuentres que vim-fugitive es un buen complemento (o no) para tu modo de trabajo. A pesar de todo, te recomendaría que le echaras un vistazo a todos los complementos que he citado anteriormente. Probablemente hay otros que no he incluido en la lista. Descúbrelos y pruébalos.

Una manera obvia de tener una mejor integración entre Vim y git es leer más sobre git. Git, por si mismo, es un tema muy amplio y aquí solo se muestra una pequeña fracción de lo que puede hacer. Con eso, vamos a *congituar* (perdón por el juego de palabras) y hablemos sobre cómo utilizar Vim para compilar tu código.
