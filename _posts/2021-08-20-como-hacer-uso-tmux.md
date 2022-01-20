---
layout: single
title: Como hacer uso de tmux
excerpt: "En este artículo, os enseño a utilizar tmux desde 0. Explicación de que es tmux, su instalación y correcta utilización."
date: 2021-08-20
classes: wide
header:
  teaser: /assets/images/como-hacer-uso-tmux/teaser.png
  teaser_home_page: true
categories:
  - Entorno de trabajo
  - Customización
  - Linux
  - Herramientas
tags:
  - tmux
  - terminal
toc: true
toc_label: "&#10148; Como hacer uso de tmux"
toc_icon: "file-alt"
toc_sticky: true
---

<style type="text/css">
    .video-container {
        position: relative;
        padding-bottom: 56.25%;
        padding-top: 35px;
        height: 0;
        overflow: hidden;
    }
    .video-container video {
        position: absolute;
        top:0;
        left: 0;
        width: 100%;
        height: 100%;
    }
</style>

`Terminal Multiplexer - tmux` es una herramienta que realiza la multiplexación de terminales. Permite lanzar `sesiones` de tmux que estas contienen `ventanas` y `paneles`. Posteriormente expondré el significado de esta nomenclatura.

Cada terminal es totalmente independiente de las demás, en cuanto a procesos que corren en ellas. La principal ventaja de tmux es la organización y productividad que ganas una vez aprendes a utilizar esta herramienta, ya que todas las funciones se pueden aplicar haciendo uso del teclado y todo lo visualizas en una única ventana de la que dispones de una sesión con diferentes ventanas y dentro de estas los diferentes paneles.

De esta manera, cuando estamos trabajando en un proyecto en el que se necesita hacer uso de varias terminales, sin tmux tendríamos cada terminal dispersa con su correspondiente ventana y para moverse entre ellas es un caos. En cambio, con tmux podemos tener diferentes espacios de trabajo con las terminales que deseemos y para movernos entre estas es mucho más sencillo.

Para que me entendáis mejor y visualicéis el uso de tmux, aquí os dejo un breve video:

<div class="video-container">
    <video autoplay muted controls allowfullscreen frameborder="0" >
        <source src="/assets/images/como-hacer-uso-tmux/uso_tmux.mp4" type="video/mp4">
    </video>
</div>

## Instalación

### tmux

Para instalar esta herramienta, como cualquier otra, depende del sistema operativo que estés corriendo en tu máquina. 

Por aquí os dejo una tabla y un link hacia la documentación oficial de la instalación de [tmux](https://github.com/tmux/tmux/wiki/Installing#from-source-tarball), por si deseáis consultar algo:


<table role="table">
    <thead>
        <tr>
            <th>Platform</th>
            <th>Install Command</th>
        </tr>
    </thead>
    <tbody>
        <tr>
            <td>Arch Linux</td>
            <td><code>pacman -S tmux</code></td>
        </tr>
        <tr>
            <td>Debian or Ubuntu</td>
            <td><code>apt install tmux</code></td>
        </tr>
        <tr>
            <td>Fedora</td>
            <td><code>dnf install tmux</code></td>
        </tr>
        <tr>
            <td>RHEL or CentOS</td>
            <td><code>yum install tmux</code></td>
        </tr>
        <tr>
            <td>macOS (using Homebrew</td>
            <td><code>brew install tmux</code></td>
        </tr>
        <tr>
            <td>macOS (using MacPorts)</td>
            <td><code>port install tmux</code></td>
        </tr>
        <tr>
            <td>openSUSE</td>
            <td><code>zypper install tmux</code></td>
        </tr>
    </tbody>
</table>


### Tema

El tema del que yo dispongo, visto previamente en el video, se denomina [`Oh my tmux`](https://github.com/gpakosz/.tmux).

Si deseáis tenerlo, únicamente debéis ejecutar los siguientes comandos desde el home de usuario `cd ~`:

```bash
$ git clone https://github.com/gpakosz/.tmux.git
$ ln -s -f .tmux/.tmux.conf
$ cp .tmux/.tmux.conf.local .
```


## Shortcuts
Una vez explicado como instalar tmux y sus ventajas, voy a explicar los shortcuts más usados para que podáis utilizar esta fantástica herramienta como nadie hasta ahora.

La gestión y nomenclatura de tmux es la siguiente:

- `Session` - Cada sesión de tmux
	- `Windows` - Dentro de esta están los paneles (panels)
		- `Panels` - Cada terminal dentro de una ventana (windows)

### Prefix
Shortcut definido para posteriormente aplicar los demás shortcuts.

`Ctrl + b` &#10132; Prefix

### Modo ratón
Este modo permite configurar la ventana activa, el panel activo, cambiar el tamaño de los paneles y cambia automáticamente al modo de copia para seleccionar texto.

`Prefix + m` &#10132; Activa el modo ratón

<div align="center">
    <img src="/assets/images/como-hacer-uso-tmux/mouse_mode.gif">
</div>

### Session

- `tmux new-session -s <name>` &#10132; Crear sesión con el nombre de esta 
- `tmux ls` or `tmux list-sessions` &#10132; Listar las sesiones
- `tmux a -t <name>` or `tmux attach -t <name>` &#10132; Abrir una sesión que esté en segundo plano
- `tmux kill-session -t <name>` &#10132; Matar una sesión

- `Prefix + $` &#10132; Cambiar el nombre
- `Prefix + d` &#10132; Establece la sesión en segundo plano, sigue corriendo todo lo que hayas dejado en las terminales de esa sesión
- `Prefix + s` &#10132; Vista general de las sesiones
- `Prefix + w` &#10132; Vista general con jerarquía - Lista las sesiones con sus respectivas ventanas y paneles


### Windown

- `Prefix + c` &#10132; Crear

- `Prefix + &` &#10132; Cerrar

- `Prefix + ,` &#10132; Cambiar nombre

- `Prefix + {1, 2, 3...}` &#10132; Cambiar entre ventanas

- `Prefix + '+'` &#10132; Mover un panel a una nueva ventana

- `Prefix + :` + `:swap-window -s <origen> -t <destino>` &#10132; Cambiar ventanas de posición


### Panel

- `Prefix + x` &#10132; Cerrar

- `Prefix + "` &#10132; Dividir horizontalmente

- `Prefix + %` &#10132; Dividir verticalmente

- `Prefix + space` &#10132; Cambiar orientación vertical u horizontal

- `Prefix + H` (izquierda) , `J` (abajo) , `K` (arriba) , `L` (derecha) &#10132; Redimensionar panel

- `Prefix + arrowkeys` &#10132; Moverse de en un panel u otro según las flechas

- `Prefix + o` &#10132; Moverse sentido agujas del reloj

- `Prefix + { or }` &#10132; Cambiar contenido entre paneles

- `Prefix + ctrl + o` &#10132; Cambiar contenido, rotación sentido contrario agujas del reloj

- `Prefix + z` &#10132; Maximizar el panel

- `Prefix + t` &#10132; Muestra la hora en el panel seleccionado

#### Modo copia y Modo selección

- `Prefix + [` &#10132; Acceder al modo copia

- `Ctrl + space` &#10132; Acceder modo selección (se debe estar previamente en el modo copia)

- `Alt + w` &#10132; Copiar

- `Prefix + ]` &#10132; Pegar lo previamente copiado