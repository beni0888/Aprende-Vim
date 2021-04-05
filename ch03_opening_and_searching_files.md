# Capítulo 3: Buscar archivos

El objetivo de este capítulo es aprender cómo buscar rápidamente en Vim. Ser capaz de buscar rápidamente es una gran manera de aumentar tu productividad en Vim. Cuando aprendí cómo buscar archivos rápidamente, fue cuando hice mi salto a Vim y a utilizarlo a todas horas.

Este capítulo está dividido en dos partes: cómo buscar sin necesidad de complementos y cómo buscar con el complemento [fzf.vim](https://github.com/junegunn/fzf.vim). Dicho esto, ¡empecemos!

## Abrir y editar archivos

Para abrir un archivo en Vim, puedes utilizar `:edit`.

```
:edit archivo.txt
```

Si `archivo.txt` existe, abre `archivo.txt` en un *buffer*. Si `archivo.txt` no existe, crea un nuevo *buffer* para `archivo.txt`.

El autocompletado del nombre con la tecla `<Tab>` funciona con `:edit`. Por ejemplo, si tu archivo está dentro de una carpeta *controller* de un proyecto en [Rails](https://rubyonrails.org/) como por ejemplo `./app/controllers/users_controllers.rb`, puedes utilizar la tecla `<Tab>` para ir expandiendo los términos rápidamente:
```
:edit a<Tab>c<Tab>u<Tab>
```

`:edit` acepta comodines como argumentos. `*` reemplaza a cualquier archivo en la carpeta actual. Si solo estás buscando archivo con la extensión `.yml` en la carpeta actual, puedes ejecutar:

```
:edit *.yml<tab>
```

Vim te mostrará una lista de todos los archivos `.yml` que existen en la carpeta actual para poder escoger el que queramos.

Puedes utilizar `**` para buscar de manera recursiva. Si quieres buscar todos los archivos `*.md` en tu proyecto, pero no estás seguro de en qué carpeta está, puedes ejecutar lo siguiente:

```
:edit **/*.md<tab>
```

`:edit` puede ser utilizado para ejecutar `netrw`, el explorador nativo de Vim. Para hacer eso, se debe dar al comando `:edit` un argumento como directorio en vez de un archivo:

```
:edit .
:edit test/unit/
```

## Buscando archivos con Find

Puedes buscar archivos con el comandos `:find`. Por ejemplo:

```
:find package.json
:find app/controllers/users_controller.rb
```

El autocompletado del nombre también funciona con el comando `:find`:

```
:find p<tab>                " Para encontrar el archivo package.json
:find a<tab>c<tab>u<tab>    " Para encontrar la ruta del archivo app/controllers/users_controller.rb
```

Quizás has notado que la sintaxis y el comportamiento del comando `:find` es similar al comando `:edit`. ¿Cuál es la diferencia?

## Find y Path

La diferencia es que `:find` encuentra un archivo en la ruta o `path`, mientras que `:edit` no lo hace.

Aprendamos un poco más sobre esta ruta o `path`. Una vez que aprendas cómo modificar tus rutas, `:find` puede convertirse en una herramienta de búsqueda muy potente. Para comprobar cuales son tus rutas, ejecuta:
```
:set path?
```

De manera predeterminada, quizás el resultado de ese comando sea algo parecido a esto:

```
path=.,/usr/include,,
```

- `.` significa buscar en relación al directorio del archivo actual.
- `,` significa buscar en el directorio actual.
- `/usr/include` es el directorio para archivos de cabeceras de compiladores de C.

Los dos primeros son importantes y el tercero puede ser ignorado por ahora. La conclusión aquí es que puedes modificar tus propias rutas. Supongamos que esta es la estructura de tu proyecto:

```
app/
  - assets/
  - controllers/
     - application_controller.rb
     - comments_controller.rb
     - users_controller.rb
  ...
```

Si quieres ir al archivo `users_controller.rb` desde el directorio raíz, debes pasar por varios directorios (y pulsar un considerable número de veces la tecla tabulador). A menudi cuando estás trabajando con un *freamework*, pasas el 90% del tiempo en un directorio en particular. En esta situalicón solo te preocupa ir al directorio `controllers/` con el mínimo de pulsaciones de teclas. Los ajustes en `path` pueden acortar ese viaje entre directorios.

Necesitas añadir `app/controllers/` al `path` actual. Aquí verás cómo puedes hacerlo:

```
:set path+=app/controllers/
```

Ahora que tu ruta ha sido actualizada, cuando escribas `:find u<Tab>`, Vim también buscará coincidencias dentro del directorio `app/controllers/` de archivos que empiezen con "u".

Si tienes un directorio `controllers/` anidado, como `app/controllers/account/users_controller.rb`, Vim no encontrará `users_controllers`. En su lugar es necesario añadir `:set path+=app/controllers/**`  así el autocompletado podrá buscar `users_controller.rb`. ¡Esto es genial! Ahora puedes encontrar lo que necesitas con solo pulsar una vez la tecla Tabulador en vez de 3 veces.

Podrás estar pensando en añadir los directorios del proyecto entero así cuando pulses `tab`, Vim buscará en cualquier lugar el archivo deseado, de esta manera:
```
:set path+=$PWD/**
```

`$PWD` hace referencia al directorio de trabajo actual. Si intentas añadir el proyecto entero a `path` para que así todos los archivos puedan ser buscado al presionar el tabulador `tab`, aunque esto puede funcionar para un proyecto pequeño, hacer esto puede ralentizar tus búsquedas de manera significativa si tienes muchos archivos en tu proyecto. Recomiendo solo añadir a `path` los directorios de los archivos más visitados.

Actualizar `path` solo te llevará unos segundos y haciendo esto te ahorrarás un montón de tiempo.

## Buscando en archivos con Grep

Si necesitas encontrar en archivos (encontrar frases en archivos), puedes usar grep. Vim tiene dos maneras de hacer esto:

- grep interno  (`:vim`. Sí, se escribe `:vim`. Es el diminutivo para `:vimgrep`).
- grep externo (`:grep`).

Revisemos el grep interno primero. `:vim` tiene la siguiente sintaxis:

```
:vim /patrón/ archivo
```

- `/patrón/` es el patrón de la expresión regular (regex) del termino buscado.
- `file` es el argumento del nombre(s) de el(los) archivo(s). De la misma forma que `:find`, también puedes pasarle los comodines `*` y `**`.

Por ejemplo, para encontrar todas las coincidencia de la cadena "desayuno" dentro de todos los archivos ruby (`.rb`) dentro de la carpeta `app/controllers/`:

```
:vim /desayuno/ app/controllers/**/*.rb
```

Después de ejecutar este comando, serás redirigido al primer resultado. El comando de búsqueda de Vim `vim` usa la operación `quickfix`. Para ver todos los resultados de la búsqueda, ejecutamos `:copen`. Esto abre una ventana `quickfix`. Aquí dejamos algunos comandos de *quickfix* útiles para empezar a trabajar:

```
:copen        Abrir la ventana quickfix
:cclose       Cerrar la ventana quickfix
:cnext        Ir al siguiente error
:cprevious    Ir al error anterior
:colder       Ir a la lista de errores mas antigua
:cnewer       Ir a la lista de errores mas nueva
```

No voy a profundizar mas en *quickfix* aquí. Para aprender mas sobre *quickfix*, revisa `:h quickfix`.

Es posible que observes que ejecutar el grep interno (`:vim`) podría ralentizarse si tienes una gran cantidad de coincidencias. Esto es provocado porque las lee desde la memoria. Vim carga cada archivo con coincidencias como si estuvieran siendo editados. Si Vim comprueba un gran número de archivos, esto consumirá una gran cantidad de memoria.

Ahora hablemos del grep externo. Por defecto, usa el comando `grep` de la terminal de comandos. Para buscar el termino "almuerzo" dentro de un archivo ruby dentro del directorio `app/controllers/`, podemos hacer lo siguiente:

```
:grep -R "almuerzo" app/controllers/
```

En vez de utilizar `/patrón`, sigue la sintáxis del comando grep para la terminal y usa `"patrón"`. También muestra todos las coincidencias usando `quickfix`.

Vim usa la variable `grepprg` para determinar qué programa externo ejecutar cuando usamos `:grep` así no tienes que usar el comando `grep` de la terminal. Posteriormente en este articulo, veremos cómo cambiar el comando externo predeterminado.

## Navegando los archivos con Netrw

`netrw` el explorador de archivos predeterminado de Vim. Es útil para ver la estructura jerárquica de un proyecto. Para poder usar `netrw`, necesitas estas 2 configuraciones en tu `.vimrc`:

```
set nocp
filetype plugin on
```

Solo veremos el uso básico de `netrw`, debería ser suficiente para empezar. Puedes iniciar `netrw` cuando inicias Vim y darle un directorio como argumento en lugar de un archivo. Por ejemplo:

```
vim .
vim src/client/
vim app/controllers/
```

Para iniciar `netrw` dentro de Vim, podemos usar `:edit` y darle un directorio como argumento directamente en lugar del nombre de un archivo:

```
:edit .
:edit src/client/
:edit app/controllers/
```

Existen otras formas de abrir una ventana de `netrw` sin darle de argumento un directorio:

```
:Explore     Inicia netrw en el archivo actual
:Sexplore    No es broma. Inicia netrw en una pantalla dividida en la mitad superior
:Vexplore    Inicia netrw en una pantalla dividida en la mitad izquierda
```

Podemos navegar dentro de `netrw` con los movimientos de Vim (veremos más sobre este tema en el capítulo 5). Si necesitas crear, eliminar, y renombrar un archivo/directorio, aquí tienes una lista de comandos `netrw` muy útiles:

```
%    Crear un nuevo archivo
d    Crear un nuevo directorio
R    Renombrar un archivo o directorio
D    Eliminar un archivo o directorio
```

`:h netrw` es muy completo. Échale un vistazo si tienes tiempo.

Si piensas que `netrw` es muy simple y necesitas más, [vim-vinegar](https://github.com/tpope/vim-vinegar) es un buen complemento para mejorar `netrw`. Si lo que buscas es un buscador de archivos diferente, [NERDTree](https://github.com/preservim/nerdtree) es una buena alternativa. ¡Pruébalos!

## FZF

Ahora que aprendimos a abrir y a buscar archivos en Vim con las herramientas incluidas, es hora de usar complementos para mejorar nuestra búsqueda.

Un acierto de los editores de texto modernos que Vim no posee es la facilidad de encontrar archivos y encontrar en los archivos. En esta segunda mitad del capítulo, te mostraré como usar [FZF.vim](https://github.com/junegunn/fzf.vim) para volver tus búsquedas en Vim mas fáciles y poderosas.

## Configuración

Pero primero, verifiquemos que tenemos [fzf](https://github.com/junegunn/fzf) y [ripgrep](https://github.com/BurntSushi/ripgrep) descargados. Para esto seguiremos las instrucciones incluidas en los respectivos repositorios de github. Los comandos `fzf` y `rg` deberían empezar a estar disponibles después de que las instalaciones de ambas herramientas fueran exitosas.

Ripgrep es una herramienta de búsqueda muy parecida a grep (de ahí el nombre). Generalmente es más rápida que grep y posee muchas características interesantes. FZF es un buscador disperso (fuzzy finder) de propósito general para la  línea de comandos. Puedes usarlo con cualquier comando incluyendo ripgrep. Juntos, hacen una poderosa herramienta de búsqueda.

FZF no usa ripgrep por defecto, así que necesitamos decirle a FZF que use ripgrep con la variable `FZF_DEFAULT_COMMAND`. En mi `.zshrc` (`.bashrc` si usas bash), deberíamos tener esto:
```
if type rg &> /dev/null; then
  export FZF_DEFAULT_COMMAND='rg --files'
  export FZF_DEFAULT_OPTS='-m'
fi
```


Debemos poner especial atención a `-m` en `FZF_DEFAULT_OPTS`. Esta opción nos permite hacer selecciones múltiples con `tab` o `shift-tab`. No es necesario tener esta línea para que FZF funcione con Vim, pero creo que es una opción muy útil que considerar. Puede volverse muy util cuando quieres realizar búsquedas y reemplazar en múltiples archivos, abarcaremos ese tema en un momento. `FZF` acepta más opciones, para aprender más, revisa [el repositorio de fzf](https://github.com/junegunn/fzf#usage) o `man fzf`. Bastará con que tengas `export FZF_DEFAULT_COMMAND='rg'`.

Después de instalar fzf y ripgrep, vamos a configurar el complemento fzf. Estoy usando [vim-plug](https://github.com/junegunn/vim-plug) como gestor de complementos en este ejemplo, pero puedes usar el que prefieras.

Tenemos que agregar lo siguiente en `.vimrc` en la sección de complementos. Necesitas usar el complemento [fzf.vim](https://github.com/junegunn/fzf.vim) (creado por el mismo autor de fzf).
```
Plug 'junegunn/fzf.vim'
Plug 'junegunn/fzf', { 'do': { -> fzf#install() } }
```

Para más información sobre este complemento, puedes revisar [fzf.Vim repo](https://github.com/junegunn/fzf/blob/master/README-VIM.md).

## La sintaxis de fzf 

Para ser capaz de utilizar fzf de manera eficiente, deberías aprender la sintaxis básica de fzf. Afortunadamente, la lista es corta:

- `^` es una coincidencia exacta de prefijo. Para buscar una frase que comience con "bienvenidos": `^bienvenidos`.
- `$` es una coincidencia exacta de sufijo. Para buscar una frase que finalice con "amigos": `amigos$`.
- `'` es una coincidencia exacta. Para buscar la frase "bienvenidos mis amigos": `'bienvenidos mis amigos`.
- `|` es una coincidencia "o". Para buscar tanto "amigos" como "enemigos": `amigos | enemigos`.
- `!` es una coincidencia inversa. Para buscar una frase que contenga "bienvenidos" y no "amigos": `bienvenidos !amigos`.

Puedes mezclar y combinar estas opciones. Por ejemplo, `^hola | ^bienvenidos amigos$` buscará la frase que comience tanto con "hola" como "bienvenidos" y que acabe con "amigos".

## Encontrar archivos

Para encontrar archivos dentro de Vim utilizando el complemento fzf.vim, puedes utilizar el método `:Files`. Ejecuta `:Files` desde Vim y se le mostrará el símbolo del sistema de la búsqueda con fzf.

<p align="center">
  <img alt="Encontrando archivos en FZF" width="900" height="auto" src="./img/fzf-files.gif"/>
</p>


Ya que utilizarás frecuentemente este comando, es una buena idea asignarle un mapeado del teclado. He mapeado el mío con `Ctrl-f`. En mi archivo `.vimrc`, he añadido lo siguiente:

```
nnoremap <silent> <C-f> :Files<CR>
```

## Encontrar en archivos
Para buscar dentro de archivos, puedes utilizar el comando `:Rg`.

<p align="center">
  <img alt="Encontrar en archivos con FZF" width="900" height="auto" src="./img/fzf-in-files.gif"/>
</p>

De nuevo, como es probable que utilices esto de manera frecuente, vamos a incluirlo en el mapeado. He creado un atajo de teclado con `<Leader>f`.

```
nnoremap <silent> <Leader>f :Rg<CR>
```

## Otras búsquedas

Fzf.vim ofrece muchos otros comandos de búsquedas. No repasaré todos de manera exahustiva, pero puedes revisarlos [aquí](https://github.com/junegunn/fzf.vim#commands).

Este es el aspecto del mapeado de mis comandos fzf:
```
nnoremap <silent> <Leader>b :Buffers<CR>
nnoremap <silent> <C-f> :Files<CR>
nnoremap <silent> <Leader>f :Rg<CR>
nnoremap <silent> <Leader>/ :BLines<CR>
nnoremap <silent> <Leader>' :Marks<CR>
nnoremap <silent> <Leader>g :Commits<CR>
nnoremap <silent> <Leader>H :Helptags<CR>
nnoremap <silent> <Leader>hh :History<CR>
nnoremap <silent> <Leader>h: :History:<CR>
nnoremap <silent> <Leader>h/ :History/<CR>
```

## Reemplazar Greo con Rg

Como he mencionado previamente, Vim tiene dos maneras de buscar en archivos: `:vim` y `:grep`. `:grep` utiliza la herramienta externa de búsqueda que puedes reasignar utilizando la palabra clave `grepprg`. Te mostraré cómo configurar Vim para utilizar ripgrep en vez del grep de la terminal al ejecutar el comando `:grep`.

Ahora configuraremos `grepprg` así `:grep` utilizará ripgrep. Añade esto en tu archivo de configuración `vimrc`.

```
set grepprg=rg\ --vimgrep\ --smart-case\ --follow
```

¡No dudes en modificar cualquiera de las opciones anteriores! Para más información de lo que significan las opciones anteriores, echa un vistazo a `man rg`.

Después de actualizar `grepprg`, ahora cuando ejecutes `:grep`, esto ahora ejecutará `rg --vimgrep --smart-case --follow` en vez de ejecutar `grep`.  Si quieres realizar una búsqueda de "donut" utilizando ripgrep, ahora puedes ejecutar el comando mencionado `:grep "donut"` en vez de `:grep "donut" . -R`.

Como ocurre con el antiguo `:grep`, este nuevo `:grep` también utiliza *quickfix* para mostrar los resultados.

Podrías preguntarte, "Bien, esto está bien, pero nunca he utilizado `:grep` en Vim, además ¿no puedo simplemente utilizar `:Rg` para encontrar frases en archivos? ¿Cuanto tendré que necesitar utilizar`:grep`?

Esta es una buena pregunta. Quizás necesites utilizar `:grep` en Vim para buscar y reemplazar en múltiples archivos, que será lo que veremos a continuación.

## Buscar y reemplazar en múltiples archivos

Los editores de texto modernos como VSCode hacen muy sencillo el buscar y reemplazar una cadena de texto en múltiples archivos a la vez. En esta sección, te mostraré dos métodos diferentes para hacer eso en Vim.

El primer método es reemplazar *todas* las frases coincidentes en tu proyecto. Necesitarás utilizar `:grep`. Si necesitas reemplazar todos las coincidencias de la palabra "pizza" y sustituirla por "donut", esto es lo que tienes que hacer:

```
:grep "pizza"
:cfdo %s/pizza/donut/g | update
```

Vamos a diseccionar los pasos que hacen estos comandos:

1. `:grep pizza` utiliza ripgrep para realizar una búsqueda de todas las instancias donde aparezca la palabra "pizza" (por cierto, esto también funcionaría incluso si no has reasignado `grepprg` para utilizar ripgrep. Deberías ejecutar `:grep "pizza" . -R` en vez de `:grep "pizza"`). 
2. `:cfdo` ejecuta cualquier comando que le asignes a continuación en la lista de *quickfix*. En este caso, tu comando es un comando de sustitución `%s/pizza/donut/g`. La tubería (`|`) es un operador de cadena. Necesitarás ejecutar `update` para guardar cada archivo después de ser sustituido. Veremos el comando de sustitución en profundidad en un capítulo posterior.

El segundo método es buscar y reemplazar en el archivo seleccionado. Con este método, puedes escoger de manera manual qué archivos seleccionar y realizar la sustitución. Esto es lo que tienes que hacer:

1. Limpiar tus *buffers*. Es imprescindible que tu lista de *buffers* contenga solo los archivos que necesitas. Puedes realizar esa limpieza mediante `:%bd | e#` (`%bd` elimina todos lod *buffers* y `e#` abre el archivo en el que estabas).
2. Ejecuta `:Files`.
3. Selecciona todos los archivos que deseas que se busque y se reemplace la cadena de texto. Para seleccionar múltiples archivos, utiliza `<Tab>` / `Shift+Tab`. Esto solo es posible si tienes la opción `-m` en `FZF_DEFAULT_OPTS` (consulta la sección anterior de ajustes de fzf para saber qué hace la opción `-m`).
4. Ejecuta `:bufdo %s/pizza/donut/g | update`. El comando `:bufdo %s/pizza/donut/g | update` tiene un aspecto similar al anterior comando `:cfdo %s/pizza/donut/g | update`. Esto es debido a que son similares. La diferencia es que en vez de realizar la sustitución en toda la lista de archivos de *quickfix* (`:cfdo`), estás realizando la sustitución en todos los *buffers* abiertos en Vim (`:bufdo`).

## Aprende a buscar de la manera más inteligente

La acción de buscar dentro de la edición de texto es una tarea imprescindible. Aprender cómo buscar de manera correcta en Vim te ayudará a mejorar tu manera de editar texto.

Fzf.vim es un elemento diferenciador. No puedo imaginar utilizar Vim sin él. Creo que es muy importante tener una buena herramienta de búsqueda al empezar con Vim. He visto a personas abandonar la transición a Vim debido a que es una de las funcionalidades modernas que tienen los editores de texto modernos y de la que Vim carece. Yo fui una de esas personas. Espero que este capítulo resuelva uno de los problemas y ayude a hacer la transición a Vim más sencilla.

También has visto en acción las posibilidades expansión de Vim, la posibilidad de poder extender las funcionalidades de búsqueda con un complemento y/o con programas externos. En el futuro, ten en cuenta qué otras funcionalidades desearías extender en Vim. Quizás alguien ha creado un complemento o ya existe un programa para ello.

Lo próximo, hablemos sobre un tema muy importante en Vim: la gramática de Vim.
