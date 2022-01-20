---
layout: single
title: Entorno profesional de trabajo en linux - bspwm
excerpt: "¿Te gustaría disponer de un entorno profesional de trabajo en linux, para ser más productivo y contar con más rendimiento en tus proyectos? Entra aqui. ¡Este es tu artículo!"
date: 2021-08-01
last_modified_at: 2021-08-12
classes: wide
header:
  teaser: /assets/images/entorno-trabajo-profesional-bspwm/entorno2.png
  teaser_home_page: true
categories:
  - Entorno de trabajo
  - Customización
  - Linux
tags:
  - bspwm
  - sxhkd
  - picom
  - rofi
  - polybar
  - linux
  - bash
toc: true
toc_label: "&#10148; Entorno de trabajo - bspwm"
toc_icon: "file-alt"
toc_sticky: true
---

En este artículo os muestro y comparto mis archivos de configuración para la creación de un entorno profesional de trabajo mediante un administrador de ventanas tipo mosaico &#10132; `bspwm`.

En primer lugar voy a dejar las especificaciones de mi sistema operativo y algunas imágenes del entorno profesional de trabajo en linux:

```bash
❯ uname -a
Linux AERO 5.8.0-63-generic #71~20.04.1-Ubuntu SMP Thu Jul 15 17:46:08 UTC 2021 x86_64 x86_64 x86_64 GNU/Linux

❯ lsb_release -a
No LSB modules are available.
Distributor ID:	Ubuntu
Description:	Ubuntu 20.04.2 LTS
Release:	20.04
Codename:	focal
```

![](/assets/images/entorno-trabajo-profesional-bspwm/entorno1.png)

![](/assets/images/entorno-trabajo-profesional-bspwm/entorno2.png)

![](/assets/images/entorno-trabajo-profesional-bspwm/entorno3.png)

Una vez visto, os voy a compartir los archivos de configuración necesarios para que podáis crearos este entorno tan chulo y hacer uso de él.

## bspwm
[bspwm](https://github.com/baskerville/bspwm) es un gestor de ventanas tipo mosaico que organiza las ventanas basándose en particiones de espacio binario. Tiene soporte para múltiples monitores.

### Archivo bspwmrc
Este es el archivo de configuración que se ejecuta cada vez que se inicia sesión con el administrador de ventanas `bspwm`. 

Para volver a ejecutarlo una vez que ya hayas iniciado sesión (por si has hecho alguna modificación), no hace falta cerrar sesión y volver a abrir sesión. Con presionar a la vez `super+alt+r` se reiniciará y se harán efecto las nuevas modificaciones.

Situado en el directorio `~/.config/bspwm/`:

```bash
#!/bin/sh

pgrep -x sxhkd > /dev/null || sxhkd &

# Workspace - Monitors (unicamente si dispones de monitores externos)
num_monitor=$(xrandr --query | grep -w connected | wc -l)
if [ $num_monitor = 3 ];then
  bspwm_monitors 3 &
  bspc monitor eDP-1 -d I II III
  bspc monitor DP-1 -d IV V VI
  bspc monitor HDMI-1-0 -d VII VIII IX X
elif [ $num_monitor = 2 ]; then
  bspwm_monitors 2 &
  bspc monitor eDP-1 -d I II III IV V
  bspc monitor DP-1 -d VI VII VIII IX X
elif [ $num_monitor = 1 ]; then
  bspwm_monitors 1 &
  bspc monitor eDP-1 -d I II III IV V VI VII VIII IX X
fi

# Descomentar si no dispones de monitores externos
# bspc monitor eDP-1 -d I II III IV V VI VII VIII IX X

wmname LG3D
xsetroot -cursor_name left_ptr &
feh --bg-fill /usr/share/backgrounds/arch_linux.png &
$HOME/.config/polybar/launch.sh &

bspc config pointer_modifier mod1
bspc config focus_follows_pointer true

bspc config border_width         2
bspc config window_gap          10

bspc config split_ratio          0.52
bspc config borderless_monocle   true
bspc config gapless_monocle      true

# Border colors
bspc config focused_border_color "#668bd7"

# Rules
bspc rule -a Org.gnome.Nautilus state=floating follow=on
bspc rule -a nm-applet state=floating
bspc rule -a Blueman-manager state=floating
bspc rule -a Pavucontrol state=floating
bspc rule -a Eog state=floating
bscp rule -a gnome-calculator state=floating
bspc rule -a file-roller state=floating

# ==================================
#               xinput
# ==================================
# Establecimiento de los botones del touchpad
#libinput Tapping Enabled (354)
xinput set-prop "FTCS1000:00 2808:0101 Touchpad" 354 1
#libinput Middle Emulation Enabled (341)
xinput set-prop "FTCS1000:00 2808:0101 Touchpad" 341 1
#libinput Natural Scrolling Enabled (334)
xinput set-prop "FTCS1000:00 2808:0101 Touchpad" 334 1

# ==================================
#           Running Apps
# ==================================
# Show window for run gui programs as root
/usr/lib/policykit-1-gnome/polkit-gnome-authentication-agent-1 &

killall nm-applet 2> /dev/null
nm-applet &

# blueman-applet
killall blueman-applet 2> /dev/null
blueman-applet &

#picom
killall picom 2> /dev/null
picom --experimental-backends &

# Dropbox
num_pid_dropbox=$(pgrep -x dropbox | wc -l)
if [ $num_pid_dropbox = 0 ]; then
	dropbox start &
fi

paplay /usr/share/sounds/Yaru/stereo/system-ready.oga --volume=30000 &
```
### Archivo bspwm_monitors
Establece las características `resolución`, `rotación`, `hz`, `prioridad` y `posición` de las pantallas (tanto la integrada, en el caso de portátiles, como las externas)

> IMPORTANTE: Modificar el contenido del archivo de tal manera.
> - Debes verificar el número de monitores y sus nombres `xrandr --query | grep connected`
> - Te ayudará el conocer el uso de la herramienta [`xrandr`](https://explainshell.com/explain/1/xrandr)

Este es el archivo de configuración, debe estar situado en un directorio al que tenga acceso la variable de entorno `PATH`. Para que así sin problema alguno, desde el fichero [`bspwmrc`](#archivo-bspwmrc) se pueda llamar al ejecutable `bspwm_monitors` junto con su respectivo argumento `1, 2 o 3`.

En mi caso elegí el directorio donde establezco mis propios ejecutables `~/.local/bin/`:

```bash
#!/bin/bash
#  ____  _   _ _  _   ____  _  ______ ___ ____  _____ 
# / ___|| | | | || | |  _ \| |/ / ___|_ _|  _ \| ____|
# \___ \| |_| | || |_| |_) | ' /\___ \| || | | |  _|  
#  ___) |  _  |__   _|  _ <| . \ ___) | || |_| | |___ 
# |____/|_| |_|  |_| |_| \_\_|\_\____/___|____/|_____|	
#
# Created by Sh4rkside

if [[ $1 = 3 ]]; then
	xrandr --output eDP-1 --mode 1920x1080 --rotate normal --rate 144.00 --output DP-1 --primary --mode 2560x1440 --rotate normal --rate 144.00 --right-of eDP-1 --output HDMI-1-0 --mode 1920x1080 --rotate normal --rate 60.00 --right-of DP-1
elif [[ $1 = 2 ]]; then
	xrandr --output eDP-1 --mode 1920x1080 --rotate normal --rate 144.00 --output DP-1 --primary --mode 2560x1440 --rotate normal --rate 144.00 --right-of eDP-1
elif [[ $1 = 1 ]]; then
	xrandr --output eDP-1 --mode 1920x1080 --rotate normal --rate 144.00 --primary
fi
```

## sxhkd
[sxhkd](https://github.com/baskerville/sxhkd) es un demonio de teclas de acceso directo para [X](https://wiki.archlinux.org/title/Xorg_(Espa%C3%B1ol)), que reacciona a los eventos de entrada ejecutando comandos.

### Archivo sxhkdrc
Este es el archivo de configuración que determina la relación entre las teclas de acceso directo junto con el comando a ejecutar.

Situado en el directorio `~/.config/sxhkd`:

```bash
#
# wm independent hotkeys
#

# terminal emulator
super + Return
	gnome-terminal

# program launcher - rofi
super + @space
	rofi -show drun -show-icons
	
super + ctrl + @space
	rofi -show run

# make sxhkd reload its configuration files:
super + Escape
	pkill -USR1 -x sxhkd

#
# bspwm hotkeys
#

# quit/restart bspwm
super + alt + {q,r}
	bspc {quit,wm -r}

# close and kill
super + {_,shift + }w
	bspc node -{c,k}

# alternate between the tiled and monocle layout
super + m
	bspc desktop -l next

# send the newest marked node to the newest preselected node
super + y
	bspc node newest.marked.local -n newest.!automatic.local

# swap the current node and the biggest window
super + g
	bspc node -s biggest.window

#
# state/flags
#

# set the window state
super + {t,shift + t,s,f}
	bspc node -t {tiled,pseudo_tiled,floating,fullscreen}

# set the node flags
super + ctrl + {m,x,y,z}
	bspc node -g {marked,locked,sticky,private}

#
# focus/swap
#

# focus the node in the given direction
super + {_,shift + }{Left,Down,Up,Right}
       bspc node -{f,s} {west,south,north,east}

# focus the node for the given path jump
super + {p,b,comma,period}
	bspc node -f @{parent,brother,first,second}

# focus the next/previous window in the current desktop
super + {_,shift + }c
	bspc node -f {next,prev}.local.!hidden.window

# focus the next/previous desktop in the current monitor
super + bracket{left,right}
	bspc desktop -f {prev,next}.local

# focus the last node/desktop
super + {grave,Tab}
	bspc {node,desktop} -f last

# focus the older or newer node in the focus history
super + {o,i}
	bspc wm -h off; \
	bspc node {older,newer} -f; \
	bspc wm -h on

# focus or send to the given desktop
super + {_,shift + }{1-9,0}
	bspc {desktop -f,node -d} '^{1-9,10}'

#
# preselect
#

# preselect the direction
super + ctrl + alt + {Left,Down,Up,Right}
	bspc node -p {west,south,north,east}

# preselect the ratio
super + ctrl + {1-9}
	bspc node -o 0.{1-9}

# cancel the preselection for the focused node
super + ctrl + space
	bspc node -p cancel

# cancel the preselection for the focused desktop
super + ctrl + shift + space
	bspc query -N -d | xargs -I id -n 1 bspc node id -p cancel

#
# move/resize
#

# move a floating window
super + ctrl + {Left,Down,Up,Right}
	bspc node -v {-20 0,0 20,0 -20,20 0}

# Custom move/resize
alt + super + {Left,Down,Up,Right}
	/home/adrianbellver/.config/bspwm/scripts/bspwm_resize {west,south,north,east}


# =================================================================
# 				CUSTOM
# ==================================================================

# Google Chrome
super + shift + g
	google-chrome-stable

# Firefox
super + shift + f
	firefox

# Files - Gnome
super + shift + e
	nautilus

# Files - Ranger
super + ctrl + e
	gnome-terminal -e 'ranger'

# Captura de pantalla
super + shift + s
	flameshot gui

# Bpytop
super + shift + b
	gnome-terminal -e 'bpytop'

# Lockscreen
super + l
	betterlockscreen -l dimblur -tf "%H:%M" -t "    Type password to unlock..."

# Ambient display
super + shift + l
	gnome-terminal -e 'unimatrix -c magenta'

# > 20% brightnessctl
ctrl + alt + m
	brightnessctl -dintel_backlight set +10%

# < 20% brightnessctl
ctrl + alt + n
	brightnessctl -dintel_backlight set 10%-
```

### Archivo bspwm_resize
Este es el archivo de configuración que determina el redimensionamiento de las ventanas que están en modo flotante. Está situado en el directorio `~/.config/bspwm/scripts`:

```bash
#!/usr/bin/env dash

if bspc query -N -n focused.floating > /dev/null; then
	step=20
else
	step=100
fi

case "$1" in
	west) dir=right; falldir=left; x="-$step"; y=0;;
	east) dir=right; falldir=left; x="$step"; y=0;;
	north) dir=top; falldir=bottom; x=0; y="-$step";;
	south) dir=top; falldir=bottom; x=0; y="$step";;
esac

bspc node -z "$dir" "$x" "$y" || bspc node -z "$falldir" "$x" "$y"
```

## picom
[picom](https://github.com/yshui/picom) es un compositor independiente para Xorg, adecuado para su utilización con administradores de ventanas que no proporcionan composición como es el caso de `bspwm`.

### Archivo picom.conf
Este es el archivo de configuración situado en el directorio `~/.config/picom`:

```bash
#################################
#             Shadows           #
#################################
shadow = false;
shadow-radius = 7;
shadow-offset-x = -7;
shadow-offset-y = -7;

shadow-exclude = [
  "name = 'Notification'",
  "class_g = 'Conky'",
  "class_g ?= 'Notify-osd'",
  "class_g = 'Cairo-clock'",
  "_GTK_FRAME_EXTENTS@:c"
];

#################################
#           Fading              #
#################################
fading = true
fade-in-step = 0.03;
fade-out-step = 0.03;

#################################
#   Transparency / Opacity      #
#################################
inactive-opacity = 0.95;
frame-opacity = 0.95;
inactive-opacity-override = false;
active-opacity = 1.0;

focus-exclude = [ "class_g = 'Cairo-clock'" ];

opacity-rule = [
  "90:class_g = 'Rofi'",
  "80:class_g = 'Nautilus'",
  #"100:class_g = 'Google-chrome'",
  "100:class_g = 'Firefox'",
  "100:class_g = 'Vmplayer'",
  "100:class_g = 'Eog'",
  "100:name *?= 'i3lock'",
  "100:name *?= 'gnome-screensaver'",
  "100:name *?= 'flameshot'"
];

#################################
#       General Settings        #
#################################

backend = "xrender";
vsync = true
mark-wmwin-focused = true;
mark-ovredir-focused = false;
detect-rounded-corners = true;
detect-client-opacity = true;
refresh-rate = 0
detect-transient = true
detect-client-leader = true
use-damage = true
log-level = "warn";

wintypes:
{
  tooltip = { fade = true; shadow = true; opacity = 0.75; focus = true; full-shadow = false; };
  dock = { shadow = false; }
  dnd = { shadow = false; }
  popup_menu = { opacity = 0.8; }
  dropdown_menu = { opacity = 0.8; }
};

corner-radius = 0;
rounded-corners-exclude = [
  "class_g = 'Polybar'",
  "class_g = 'Google-chrome'",
  "class_g = 'firefox'"
];
```

## rofi
[rofi](https://github.com/davatorium/rofi) es un selector de ventanas y un lanzador de aplicaciones.

### Archivo config.rasi
Este es el archivo de configuración que determina el tema que se le aplica a rofi. Se puede importar cualquiera de los temas que se sitúen bajo el directorio `/usr/share/rofi/themes`.

Situado en el directorio `~/.config/rofi`:

```css
@import "/usr/share/rofi/themes/Arc-Dark.rasi"
```

### Menú powermenu
---

Menú funcional para llevar acabo la automatización de varias tareas, en concreto `apagar`, `reinicar`, `suspender`, `bloquear la pantalla` o `cerrar sesión`. Es accesible desde la polybar y esta desarrollado bajo bash y rofi.

<p align="center">
    <img src="/assets/images/entorno-trabajo-profesional-bspwm/powermenu1.png">
</p>

<p align="center">
    <img src="/assets/images/entorno-trabajo-profesional-bspwm/powermenu2.png">
</p>

#### powermenu.sh
Este es el script que hace uso el módulo `powermenu` definido en el archivo de configuración de la polybar [`config`](#archivo-config).

Situado en el directorio `~/.config/rofi/powermenu`:

```bash
#!/usr/bin/env bash

theme="setting"
dir="$HOME/.config/rofi/powermenu"
user=$(whoami)

rofi_command="rofi -theme $dir/$theme"

# Options
shutdown=" Shutdown"
reboot=" Reboot"
suspend=" Suspend"
lock=" Lockscreen"
logout=" Logout"

# Confirmation
confirm_exit() {
	rofi -dmenu\
		-i\
		-no-fixed-num-lines\
		-p "Are You Sure?"\
		-theme $dir/confirm.rasi
}

# Variable passed to rofi
options="$shutdown\n$reboot\n$suspend\n$lock\n$logout"

chosen="$(echo -e "$options" | $rofi_command -p "$user" -dmenu -selected-row 2)"
case $chosen in
    $shutdown)
        ans=$(confirm_exit &)
        if [[ $ans == "y" || $ans == "Y" ]]; then
            systemctl poweroff
        elif [[ $ans == "n" || $ans == "N" ]]; then
            exit 0
        fi
        ;;
    $reboot)
        ans=$(confirm_exit &)
        if [[ $ans == "y" || $ans == "Y" ]]; then
            systemctl reboot
        elif [[ $ans == "n" || $ans == "N" ]]; then
            exit 0
        fi
        ;;
    $lock)
        if [[ -f /home/adrianbellver/.local/bin/betterlockscreen ]]; then
            betterlockscreen -l dimblur --time-format "%H:%M"
        fi
        ;;
    $suspend)
        ans=$(confirm_exit &)
        if [[ $ans == "y" || $ans == "Y" ]]; then
            betterlockscreen -l dimblur --time-format "%H:%M" &
            sleep 2s
            systemctl suspend
        elif [[ $ans == "n" || $ans == "N" ]]; then
            exit 0
        fi
        ;;
    $logout)
        ans=$(confirm_exit &)
        if [[ $ans == "y" || $ans == "Y" ]]; then
            if [[ "$DESKTOP_SESSION" == "bspwm" ]]; then
                bspc quit
            fi
        elif [[ $$ans == "n" || $ans == "N" ]]; then
            exit 0
        fi
        ;;
esac
```
#### setting.rasi
Este es el archivo de configuración que da el aspecto al menú principal.

Situado en el directorio `~/.config/rofi/powermenu`:
```bash
configuration {
    display-drun:                   "app";
    display-window:                 "window";
    show-icons:true;
}
 
* {
    lines: 10;
    columns: 1;
    border-color: #668bd7;
    text-color: #0c051f;
    font: "Roboto 11";
}
 
#window {
    border: 2;
    border-radius: 4px;
    padding: 0px;
    width: 25%;
    height: 21.50%;
}
 
#mainbox {
    background-color: #0c051f00;
    children: [inputbar, listview];
    spacing: 10px;
    padding: 0 0 0 0;
    border: 15px;
    border-color: #0c051f00;
}
 
 
#listview {
    background-color: #0c051f00;
    fixed-height: 0;
    border: 0px;
    spacing: 10px;
    scrollbar: false;
    padding: 0px 0px 0px;
}
 
#element {
    background-color: #0c051f00;
    border: 0;
    border-radius: 30px;
    padding: 4 4 4 4 ;
}
 
#element selected {
    background-color: #DE652E;
    text-color: #f1d389;
}
 
 
#inputbar {
    children:   [ prompt,textbox-prompt-colon,entry,case-indicator ];
    background-color: #0c051f00;
}
 
#case-indicator {
    background-color: #0c051f00;
    spacing:    0;
}
#entry {
    background-color: #0c051f00;
    spacing:    0;
}
#prompt {
    background-color: #0c051f00;
    spacing:    0;
}
 
#textbox-prompt-colon {
    background-color: #0c051f00;
    expand:     false;
    str:        ":";
    margin:     0px 0.3em 0em 0em ;
}
```

#### confirm.rasi
Este es el archivo de configuración que da el aspecto al menú de confirmación.

Situado en el directorio `~/.config/rofi/powermenu`:

```bash
configuration {
    display-drun:                   "app";
    display-window:                 "window";
    show-icons:true;
}
 
* {
    lines: 10;
    columns: 1;
    border-color: #668bd7;
    text-color: #0c051f;
    font: "Roboto 11";
}
 
#window {
    border: 2;
    border-radius: 4px;
    padding: 0px;
    width: 10%;
    height: 5.50%;
}
 
#mainbox {
    background-color: #0c051f00;
    children: [inputbar, listview];
    spacing: 10px;
    padding: 0 0 0 0;
    border: 15px;
    border-color: #0c051f00;
}
 
 
#listview {
    background-color: #0c051f00;
    fixed-height: 0;
    border: 0px;
    spacing: 10px;
    scrollbar: false;
    padding: 0px 0px 0px;
}
 
#element {
    background-color: #0c051f00;
    border: 0;
    border-radius: 30px;
    padding: 4 4 4 4 ;
}
 
#element selected {
    background-color: #DE652E;
    text-color: #f1d389;
}
 
 
#inputbar {
    children:   [ prompt,textbox-prompt-colon,entry,case-indicator ];
    background-color: #0c051f00;
}
 
#case-indicator {
    background-color: #0c051f00;
    spacing:    0;
}
#entry {
    background-color: #0c051f00;
    spacing:    0;
}
#prompt {
    background-color: #0c051f00;
    spacing:    0;
}
 
#textbox-prompt-colon {
    background-color: #0c051f00;
    expand:     false;
    str:        ":";
    margin:     0px 0.3em 0em 0em ;
}
```

## polybar
[polybar](https://github.com/polybar/polybar) es una herramienta rápida y totalmente customizable para crear barras de estado.

### Archivo launch.sh
Este es el archivo de configuración que lanza las barras de estado que se definen en el archivo [`config`](#archivo-config).

Situado en el directorio `~/.config/polybar`:

```bash
#!/usr/bin/env bash

# Terminate already running bar instances
killall -q polybar

# Wait until the processes have been shut down
while pgrep -u $UID -x polybar >/dev/null; do sleep 1; done

for m in $(xrandr --query | grep -w connected | awk '{print $1}'); do
	MONITOR=$m polybar -c $HOME/.config/polybar/config --reload mainbar-bspwm &
done

echo "Bars launched..."
```

### Archivo config
Este es el archivo de configuración situado en el directorio `~/.config/polybar`:

```bash
#  ____  _   _ _  _   ____  _  ______ ___ ____  _____ 
# / ___|| | | | || | |  _ \| |/ / ___|_ _|  _ \| ____|
# \___ \| |_| | || |_| |_) | ' /\___ \| || | | |  _|  
#  ___) |  _  |__   _|  _ <| . \ ___) | || |_| | |___ 
# |____/|_| |_|  |_| |_| \_\_|\_\____/___|____/|_____|	
#
# Created by Sh4rkside

;=====================================================
;
; To learn more about how to configure Polybar
; go to https://github.com/jaagr/polybar
;
; The README contains alot of information
; Themes : https://github.com/jaagr/dots/tree/master/.local/etc/themer/themes
; https://github.com/jaagr/polybar/wiki/
; https://github.com/jaagr/polybar/wiki/Configuration
; https://github.com/jaagr/polybar/wiki/Formatting
;
;=====================================================

[global/wm]
;https://github.com/jaagr/polybar/wiki/Configuration#global-wm-settings
margin-top = 0
margin-bottom = 0

[settings]
;https://github.com/jaagr/polybar/wiki/Configuration#application-settings
throttle-output = 5
throttle-output-for = 10
screenchange-reload = true
compositing-background = over
compositing-foreground = over
compositing-overline = over
compositing-underline = over
compositing-border = over

; Define fallback values used by all module formats
format-foreground = #FF0000
format-background = #00FF00
format-underline =
format-overline =
format-spacing =
format-padding =
format-margin =
format-offset =

[colors]
; Nord theme ============
background = #282c34
foreground = #abb2bf
alert = #bd2c40
volume-min = #a3be8c
volume-med = #ebcb8b
volume-max = #bf616a
; =======================

; Gotham theme ==========
; background = #0a0f14
; foreground = #99d1ce
; alert = #d26937
; volume-min = #2aa889
; volume-med = #edb443
; volume-max = #c23127
; =======================

; INTRCPTR theme ============
;background = ${xrdb:color0:#222}
;background = #aa000000
;background-alt = #444
;foreground = ${xrdb:color7:#222}
;foreground = #fff
;foreground-alt = #555
;primary = #ffb52a
;secondary = #e60053
;alert = #bd2c40

################################################################################
################################################################################
############                  MAINBAR-BSPWM                         ############
################################################################################
################################################################################

[bar/mainbar-bspwm]
monitor = ${env:MONITOR}
;monitor-fallback = HDMI1
width = 100%
height = 20
;offset-x = 1%
;offset-y = 1%
radius = 0.0
fixed-center = true
bottom = false
separator = 

background = ${colors.background}
foreground = ${colors.foreground}

line-size = 2
line-color = #f00

wm-restack = bspwm
override-redirect = false

; Enable support for inter-process messaging
; See the Messaging wiki page for more details.
enable-ipc = true

border-size = 0
;border-left-size = 0
;border-right-size = 25
;border-top-size = 0
;border-bottom-size = 25
border-color = #00000000

padding-left = 1
padding-right = 1

module-margin-left = 0
module-margin-right = 0

;https://github.com/jaagr/polybar/wiki/Fonts
font-0 = "UbuntuMono Nerd Font:size=10;2"
font-1 = "UbuntuMono Nerd Font:size=16;3"
font-2 = "Font Awesome 5 Free Regular:style=Regular:pixelsize=8;1"
font-3 = "Font Awesome 5 Free Solid:style=Solid:pixelsize=8;1"
font-4 = "Font Awesome 5 Brands Regular:style=Regular:pixelsize=8;1"
font-5 = "Hack Nerd Font:style=Regular:pixelsize=16;4"

modules-left = bspwm xwindow
modules-center = 
modules-right = xkeyboard arrow_morado_f uptime arrow2 memory3 arrow3 cpu2 temperaturecpu arrow2 wireless-network arrow3 wired-network arrow2 pulseaudio-control arrow3 date arrow2 backlight arrow3 battery arrow2 powermenu

tray-detached = false
tray-offset-x = 0
tray-offset-y = 0
tray-position = right
tray-padding = 2
tray-maxsize = 13
tray-scale = 1.0
tray-background = #668bd7
;tray-foreground = #fefefe

scroll-up = bspwm-desknext
scroll-down = bspwm-deskprev

################################################################################
################################################################################
############                       MODULE BSPWM                     ############
################################################################################
################################################################################

[module/bspwm]
type = internal/bspwm

enable-click = true
enable-scroll = true
reverse-scroll = true
pin-workspaces = true

ws-icon-0 = I;
ws-icon-1 = II;
ws-icon-2 = III;
ws-icon-3 = IV;
ws-icon-4 = V;
ws-icon-5 = VI;
ws-icon-6 = VII;
ws-icon-7 = VIII;
ws-icon-8 = IX;
ws-icon-9 = X;
ws-icon-default = " "


format = <label-state> <label-mode>

label-focused = %icon%
label-focused-background = ${colors.background}
label-focused-foreground = #8d62ad
label-focused-underline= #8d62ad
label-focused-padding = 2
;label-focused-foreground = ${colors.foreground}

label-occupied = %icon%
label-occupied-foreground = #668bd7
label-occupied-padding = 2
label-occupied-background = ${colors.background}

label-urgent = %icon%
label-urgent-padding = 2

label-empty = %icon%
label-empty-foreground = ${colors.foreground}
label-empty-padding = 2
label-empty-background = ${colors.background}
label-monocle = "  "
label-monocle-foreground = ${colors.foreground}
label-tiled = "  "
label-tiled-foreground = ${colors.foreground}
label-fullscreen = "  "
label-fullscreen-foreground = ${colors.foreground}
label-floating = "  "
label-floating-foreground = ${colors.foreground}
label-pseudotiled = "  "
label-pseudotiled-foreground = ${colors.foreground}
label-locked = "  "
label-locked-foreground = ${colors.foreground}
label-sticky = "  "
label-sticky-foreground = ${colors.foreground}
label-private =  "     "
label-private-foreground = ${colors.foreground}

; Separator in between workspaces
;label-separator = |
;label-separator-padding = 10
;label-separator-foreground = #ffb52a

format-foreground = ${colors.foreground}
format-background = ${colors.background}




################################################################################
###############################################################################
############                       MODULES ARROWS                     ############
################################################################################
################################################################################

[module/arrow1]
; grey to Blue
type = custom/text
content = "%{T2} %{T-}"
content-font = 5
content-foreground = #8d62a9
content-background = #292d3e

[module/arrow2]
; grey to Blue
type = custom/text
content = "%{T2} %{T-}"
content-font = 5
content-foreground = #668bd7
content-background = #8d62a9

[module/arrow3]
; grey to Blue
type = custom/text
content = "%{T2} %{T-}"
content-font = 5
content-foreground = #8b62a9
content-background = #668bd7

[module/arrow_morado_f]
type = custom/text
content = "%{T2} %{T-}"
content-font = 5
content-foreground = #8b62a9
content-background = #282c34

[module/arrow_azul_f]
type = custom/text
content = "%{T2} %{T-}"
content-font = 5
content-foreground = #668bd7
content-background = #282c34

################################################################################
###############################################################################
#                      					MODULES                      		
################################################################################
################################################################################

################################################################################
[module/temperaturecpu]
type = custom/script
exec = sensors | grep 'Package id' | cut -d' ' -f5 | sed 's/+//' | sed 's/.$//'
tail = false
interval = 5

format-background = #8d62ad
format-foreground = #fefefe

format-prefix = "  "
format-prefix-foreground = #fefefe

################################################################################

[module/powermenu]
type = custom/text
content = "  "

content-background = #668bd7
content-foreground = #fefefe

click-left = ~/.config/rofi/powermenu/powermenu.sh &
click-right = bash /opt/google/chrome/google-chrome --profile-directory=Default --app-id=kjbdgfilnfhdoflbpgamdcdgpehopbep


################################################################################

[module/backlight]
;https://github.com/jaagr/polybar/wiki/Module:-backlight

type = internal/backlight

; Use the following command to list available cards:
; $ ls -1 /sys/class/backlight/
card = intel_backlight

; Available tags:
;   <label> (default)
;   <ramp>
;   <bar>
format = <ramp> <label>
format-foreground = #fefefe
format-background = #668bd7

; Available tokens:
;   %percentage% (default)
label = %percentage%%

; Only applies if <ramp> is used
ramp-0 = " 🌕"
ramp-1 = " 🌔"
ramp-2 = " 🌓"
ramp-3 = " 🌒"
ramp-4 = " 🌑"

; Only applies if <bar> is used
bar-width = 10
bar-indicator = |
bar-fill = ─
bar-empty = ─

################################################################################

[module/battery]
;https://github.com/jaagr/polybar/wiki/Module:-battery
type = internal/battery
battery = BAT1
adapter = ADP1
full-at = 100

format-charging = <animation-charging> <label-charging>
label-charging =  %percentage%%
;format-charging-underline = #a3c725
format-charging-foreground = #fefefe
format-charging-background = #8d62ad
;format-charging-foreground = ${colors.foreground}
;format-charging-background = ${colors.background}

format-discharging = <ramp-capacity> <label-discharging>
label-discharging =  %percentage%%
;format-discharging-underline = #c7ae25
format-discharging-foreground = #fefefe
format-discharging-background = #8d62ad
;format-discharging-foreground = ${colors.foreground}
;format-discharging-background = ${colors.background}

format-full-prefix = " "
format-full-prefix-foreground = #a3c725
format-full-underline = #a3c725
format-full-foreground = #fefefe
format-full-background = #8d62ad
;format-full-foreground = ${colors.foreground}
;format-full-background = ${colors.background}

ramp-capacity-0 = 
ramp-capacity-1 = 
ramp-capacity-2 = 
ramp-capacity-3 = 
ramp-capacity-4 = 
ramp-capacity-foreground = #c7ae25

animation-charging-0 = 
animation-charging-1 = 
animation-charging-2 = 
animation-charging-3 = 
animation-charging-4 = 
animation-charging-foreground = #a3c725
animation-charging-framerate = 750

################################################################################

[module/cpu2]
;https://github.com/jaagr/polybar/wiki/Module:-cpu
type = internal/cpu
; Seconds to sleep between updates
; Default: 1
interval = 1
format-foreground = #fefefe
format-background = #8d62ad
format-prefix = "  "
format-prefix-foreground = #fefefe

label-font = 1

; Available tags:
;   <label> (default)
;   <bar-load>
;   <ramp-load>
;   <ramp-coreload>
format = <label>


; Available tokens:
;   %percentage% (default) - total cpu load
;   %percentage-cores% - load percentage for each core
;   %percentage-core[1-9]% - load percentage for specific core
;label = cpu %percentage:3%%
label = %percentage%% |


################################################################################

[module/date]
;https://github.com/jaagr/polybar/wiki/Module:-date
type = internal/date
; Seconds to sleep between updates
interval = 5
; See "http://en.cppreference.com/w/cpp/io/manip/put_time" for details on how to format the date string
; NOTE: if you want to use syntax tags here you need to use %%{...}
date = " %a %b %d, %Y"
date-alt = " %a %b %d, %Y"
time = %H:%M
time-alt = %l:%M%p
format-prefix = " "
format-prefix-foreground = #fefefe
format-foreground = #fefefe
format-background = #8d62ad
label = "%date% %time% "

################################################################################

[module/memory3]
;https://github.com/jaagr/polybar/wiki/Module:-memory
type = internal/memory
interval = 1
; Available tokens:
;   %percentage_used% (default)
;   %percentage_free%
;   %gb_used%
;   %gb_free%
;   %gb_total%
;   %mb_used%
;   %mb_free%
;   %mb_total%
label = %gb_used%

format = <label>
format-prefix = "  "
format-prefix-foreground = #fefefe
format-foreground = #fefefe
format-background = #668bd7

################################################################################

[module/pulseaudio-control]
type = custom/script
tail = true
;label = %output%
label-padding = 2
exec = pulseaudio-control --autosync --format '$VOL_ICON ${VOL_LEVEL}% "|" $SINK_NICKNAME' --sink-nickname 'alsa_output.pci-0000_00_1f.3.analog-stereo:' --sink-nickname 'alsa_output.pci-0000_01_00.1.hdmi-stereo:﴿' --sink-nickname 'bluez_sink.E4_41_22_2A_A2_CF.a2dp_sink:' --color-muted ffffff --icons-volume " , " --icon-muted " " listen
click-right = exec pavucontrol &
click-left = paplay /usr/share/sounds/Yaru/stereo/bell.oga; pulseaudio-control togmute
click-middle = pulseaudio-control next-sink; paplay /usr/share/sounds/Yaru/stereo/bell.oga
scroll-up = pulseaudio-control --volume-step 5 --volume-max 153 up; paplay /usr/share/sounds/Yaru/stereo/audio-volume-change.oga
scroll-down = pulseaudio-control --volume-step 5 down; paplay /usr/share/sounds/Yaru/stereo/audio-volume-change.oga
format-foreground = #fefefe
format-background = #668bd7

################################################################################

[module/uptime]
;https://github.com/jaagr/polybar/wiki/User-contributed-modules#uptime
type = custom/script
exec = uptime | awk -F, '{sub(".*up ",x,$1);print $1}'
interval = 100
label = %output%

format-foreground = #fefefe
format-background = #8d62ad
format-prefix = "  "
;format-prefix-foreground = #C15D3E
;format-underline = #C15D3E

#################################################################################

[module/wired-network]
;https://github.com/jaagr/polybar/wiki/Module:-network
type = internal/network
interface = enp2s0
;interface = enp14s0
interval = 3.0

; Available tokens:
;   %ifname%    [wireless+wired]
;   %local_ip%  [wireless+wired]
;   %essid%     [wireless]
;   %signal%    [wireless]
;   %upspeed%   [wireless+wired]
;   %downspeed% [wireless+wired]
;   %linkspeed% [wired]
; Default: %ifname% %local_ip%
label-connected =  %ifname% | %upspeed:7% %downspeed:7% | %local_ip%

label-disconnected = %ifname%

format-connected-foreground = #fefefe
format-connected-background = #8d62ad
;format-connected-underline = #55aa55
format-connected-prefix = "  "
format-connected-prefix-foreground = #55aa55
format-connected-prefix-background = #8d62ad

format-disconnected-foreground = #fefefe
format-disconnected-background = #8d62ad
;format-disconnected-underline = #FF0000
format-disconnected-prefix = "  "
format-disconnected-prefix-foreground = #FF0000
format-disconnected-prefix-background = #8d62ad

;format-disconnected-prefix = " "
;format-disconnected-prefix-foreground = #55aa55
;format-disconnected-prefix-background = #8d62ad
;format-disconnected = <label-disconnected>
;format-disconnected-underline = ${colors.alert}
;label-disconnected-foreground = #fefefe

################################################################################

[module/wireless-network]
;https://github.com/jaagr/polybar/wiki/Module:-network
type = internal/network
interface = wlo1
interval = 3.0
label-connected = %essid% - %signal% | %upspeed:7% %downspeed:7% | %local_ip%
format-connected = <label-connected> 
;format-connected = <ramp-signal> <label-connected>
format-connected-foreground = #fefefe
format-connected-background = #668bd7
format-connected-prefix = "  "
format-connected-prefix-foreground = #55aa55
format-connected-prefix-background = #668bd7
;format-connected-underline = #7e52c6

label-disconnected = %ifname%
label-disconnected-foreground = #fefefe
label-disconnected-background = #668bd7

format-disconnected = <label-disconnected>
format-disconnected-foreground = #fefefe
format-disconnected-background = #668bd7
format-disconnected-prefix = "  "
format-disconnected-prefix-foreground = #FF0000
format-disconnected-prefix-background = #668bd7
;format-disconnected-underline =${colors.alert}

ramp-signal-0 = ▁
ramp-signal-1 = ▂
ramp-signal-2 = ▃
ramp-signal-3 = ▄
ramp-signal-4 = ▅
ramp-signal-5 = ▆
ramp-signal-6 = ▇
ramp-signal-7 = █
ramp-signal-foreground = #7e52c6

################################################################################

[module/xkeyboard]
;Muestra unicamente los iconos cuando estan activadas las funciones (Caps y Num lock)
;https://github.com/jaagr/polybar/wiki/Module:-xkeyboard
type = internal/xkeyboard
blacklist-0 = scroll lock

;format-prefix = " "
;format-prefix-foreground = ${colors.foreground}
;format-prefix-background = ${colors.background}
;format-prefix-underline = #3ecfb2
format-foreground = ${colors.foreground}
format-background = ${colors.background}

label-indicator-on-capslock = " "
;Las siguientes variables estan mal definidas => Por ello establezco como string vacio - on-numlock
label-indicator-on-numlock = ""
label-indicator-off-numlock = " "
;
label-indicator-padding = 0
label-indicator-margin = 1
label-layout = ""
;label-layout-underline = #3ecfb2


################################################################################

[module/xwindow]
;https://github.com/jaagr/polybar/wiki/Module:-xwindow
type = internal/xwindow

; Available tokens:
;   %title%
; Default: %title%
label = %title%
label-maxlen = 40

format-prefix = "  "
format-prefix-underline = #292d3e
format-underline = #e1acff
format-foreground = #e1acff
format-background = ${colors.background}

###############################################################################
```

### Archivo pulseaudio-control
Este es el script que hace uso el módulo `pulseaudio-control` definido en el archivo de configuración de la polybar [`config`](#archivo-config). Sirve para controlar las salidas de audio y el volumen de estas.

Situado en el directorio donde establezco mis propios ejecutables `~/.local/bin/`:

```bash
#!/usr/bin/env bash

# Defaults for configurable values, expected to be set by command-line arguments
AUTOSYNC="no"
COLOR_MUTED="%{F#6b6b6b}"
ICON_MUTED=
ICON_SINK=
NOTIFICATIONS="yes"
OSD="no"
SINK_NICKNAMES_PROP=
VOLUME_STEP=2
VOLUME_MAX=130
# shellcheck disable=SC2016
FORMAT='$VOL_ICON ${VOL_LEVEL}% $ICON_SINK $SINK_NICKNAME'
declare -A SINK_NICKNAMES
declare -a ICONS_VOLUME
declare -a SINK_BLACKLIST

# Environment & global constants for the script
export LANG=en_US  # Some calls depend on English outputs of pactl
END_COLOR="%{F-}"  # For Polybar colors


# Saves the currently default sink into a variable named `curSink`. It will
# return an error code when pulseaudio isn't running.
function getCurSink() {
    if ! pactl info &>/dev/null; then return 1; fi
    local curSinkName

    curSinkName=$(pactl info | awk '/Default Sink: / {print $3}')
    curSink=$(pactl list sinks | grep -B 4 -E "Name: $curSinkName\$" | sed -nE 's/^Sink #([0-9]+)$/\1/p')
}


# Saves the sink passed by parameter's volume into a variable named `VOL_LEVEL`.
function getCurVol() {
    VOL_LEVEL=$(pactl list sinks | grep -A 15 -E "^Sink #$1\$" | grep 'Volume:' | grep -E -v 'Base Volume:' | awk -F : '{print $3; exit}' | grep -o -P '.{0,3}%' | sed 's/.$//' | tr -d ' ')
}


# Saves the name of the sink passed by parameter into a variable named
# `sinkName`.
function getSinkName() {
    sinkName=$(pactl list sinks short | awk -v sink="$1" '{ if ($1 == sink) {print $2} }')
    portName=$(pactl list sinks | grep -e 'Sink #' -e 'Active Port: ' | sed -n "/^Sink #$1\$/,+1p" | awk '/Active Port: / {print $3}')
}


# Saves the name to be displayed for the sink passed by parameter into a
# variable called `SINK_NICKNAME`.
# If a mapping for the sink name exists, that is used. Otherwise, the string
# "Sink #<index>" is used.
function getNickname() {
    getSinkName "$1"
    unset SINK_NICKNAME

    if [ -n "$sinkName" ] && [ -n "$portName" ] && [ -n "${SINK_NICKNAMES[$sinkName/$portName]}" ]; then
        SINK_NICKNAME="${SINK_NICKNAMES[$sinkName/$portName]}"
    elif [ -n "$sinkName" ] && [ -n "${SINK_NICKNAMES[$sinkName]}" ]; then
        SINK_NICKNAME="${SINK_NICKNAMES[$sinkName]}"
    elif [ -n "$sinkName" ] && [ -n "$SINK_NICKNAMES_PROP" ]; then
        getNicknameFromProp "$SINK_NICKNAMES_PROP" "$sinkName"
        # Cache that result for next time
        SINK_NICKNAMES["$sinkName"]="$SINK_NICKNAME"
    fi

    if [ -z "$SINK_NICKNAME" ]; then
        SINK_NICKNAME="Sink #$1"
    fi
}

# Gets sink nickname based on a given property.
function getNicknameFromProp() {
    local nickname_prop="$1"
    local for_name="$2"

    SINK_NICKNAME=
    while read -r property value; do
        case "$property" in
            Nombre:)
                sink_name="$value"
                unset sink_desc
                ;;
            "$nickname_prop")
                if [ "$sink_name" != "$for_name" ]; then
                    continue
                fi
                SINK_NICKNAME="${value:3:-1}"
                break
                ;;
        esac
    done < <(pactl list sinks)
}

# Saves the status of the sink passed by parameter into a variable named
# `isMuted`.
function getIsMuted() {
    isMuted=$(pactl list sinks | grep -E "^Sink #$1\$" -A 15 | awk '/Mute: / {print $2}')
}


# Saves all the sink inputs of the sink passed by parameter into a string
# named `sinkInputs`.
function getSinkInputs() {
    sinkInputs=$(pactl list sink-inputs | grep -B 4 "Sink: $1" | sed -nE 's/^Sink Input #([0-9]+)$/\1/p')
}


function volUp() {
    # Obtaining the current volume from pulseaudio into $VOL_LEVEL.
    if ! getCurSink; then
        echo "PulseAudio not running"
        return 1
    fi
    getCurVol "$curSink"
    local maxLimit=$((VOLUME_MAX - VOLUME_STEP))

    # Checking the volume upper bounds so that if VOLUME_MAX was 100% and the
    # increase percentage was 3%, a 99% volume would top at 100% instead
    # of 102%. If the volume is above the maximum limit, nothing is done.
    if [ "$VOL_LEVEL" -le "$VOLUME_MAX" ] && [ "$VOL_LEVEL" -ge "$maxLimit" ]; then
        pactl set-sink-volume "$curSink" "$VOLUME_MAX%"
    elif [ "$VOL_LEVEL" -lt "$maxLimit" ]; then
        pactl set-sink-volume "$curSink" "+$VOLUME_STEP%"
    fi

    if [ $OSD = "yes" ]; then showOSD "$curSink"; fi
    if [ $AUTOSYNC = "yes" ]; then volSync; fi
}


function volDown() {
    # Pactl already handles the volume lower bounds so that negative values
    # are ignored.
    if ! getCurSink; then
        echo "PulseAudio not running"
        return 1
    fi
    pactl set-sink-volume "$curSink" "-$VOLUME_STEP%"

    if [ $OSD = "yes" ]; then showOSD "$curSink"; fi
    if [ $AUTOSYNC = "yes" ]; then volSync; fi
}


function volSync() {
    if ! getCurSink; then
        echo "PulseAudio not running"
        return 1
    fi
    getSinkInputs "$curSink"
    getCurVol "$curSink"

    # Every output found in the active sink has their volume set to the
    # current one. This will only be called if $AUTOSYNC is `yes`.
    for each in $sinkInputs; do
        pactl set-sink-input-volume "$each" "$VOL_LEVEL%"
    done
}


function volMute() {
    # Switch to mute/unmute the volume with pactl.
    if ! getCurSink; then
        echo "PulseAudio not running"
        return 1
    fi
    if [ "$1" = "toggle" ]; then
        getIsMuted "$curSink"
        if [ "$isMuted" = "yes" ]; then
            pactl set-sink-mute "$curSink" "no"
        else
            pactl set-sink-mute "$curSink" "yes"
        fi
    elif [ "$1" = "mute" ]; then
        pactl set-sink-mute "$curSink" "yes"
    elif [ "$1" = "unmute" ]; then
        pactl set-sink-mute "$curSink" "no"
    fi

    if [ $OSD = "yes" ]; then showOSD "$curSink"; fi
}


function nextSink() {
    # The final sinks list, removing the blacklisted ones from the list of
    # currently available sinks.
    if ! getCurSink; then
        echo "PulseAudio not running"
        return 1
    fi

    # Obtaining a tuple of sink indexes after removing the blacklisted devices
    # with their name.
    sinks=()
    local i=0
    while read -r line; do
        index=$(echo "$line" | cut -f1)
        name=$(echo "$line" | cut -f2)

        # If it's in the blacklist, continue the main loop. Otherwise, add
        # it to the list.
        for sink in "${SINK_BLACKLIST[@]}"; do
            if [ "$sink" = "$name" ]; then
                continue 2
            fi
        done

        sinks[$i]="$index"
        i=$((i + 1))
    done < <(pactl list short sinks | sort -n)

    # If the resulting list is empty, nothing is done
    if [ ${#sinks[@]} -eq 0 ]; then return; fi

    # If the current sink is greater or equal than last one, pick the first
    # sink in the list. Otherwise just pick the next sink avaliable.
    local newSink
    if [ "$curSink" -ge "${sinks[-1]}" ]; then
        newSink=${sinks[0]}
    else
        for sink in "${sinks[@]}"; do
            if [ "$curSink" -lt "$sink" ]; then
                newSink=$sink
                break
            fi
        done
    fi

    # The new sink is set
    pactl set-default-sink "$newSink"

    # Move all audio threads to new sink
    local inputs
    inputs="$(pactl list short sink-inputs | cut -f 1)"
    for i in $inputs; do
        pactl move-sink-input "$i" "$newSink"
    done

    if [ $NOTIFICATIONS = "yes" ]; then
        getNickname "$newSink"

        if command -v dunstify &>/dev/null; then
            notify="dunstify --replace 201839192"
        else
            notify="notify-send"
        fi
        $notify "PulseAudio" "Changed output to $SINK_NICKNAME" --icon=audio-headphones-symbolic &
    fi
}


# This function assumes that PulseAudio is already running. It only supports
# KDE OSDs for now. It will show a system message with the status of the
# sink passed by parameter, or the currently active one by default.
function showOSD() {
    if [ -z "$1" ]; then
        curSink="$1"
    else
        getCurSink
    fi
    getCurVol "$curSink"
    getIsMuted "$curSink"
    qdbus org.kde.kded /modules/kosd showVolume "$VOL_LEVEL" "$isMuted"
}


function listen() {
    local firstRun=0

    # Listen for changes and immediately create new output for the bar.
    # This is faster than having the script on an interval.
    pactl subscribe 2>/dev/null | {
        while true; do
            {
                # If this is the first time just continue and print the current
                # state. Otherwise wait for events. This is to prevent the
                # module being empty until an event occurs.
                if [ $firstRun -eq 0 ]; then
                    firstRun=1
                else
                    read -r event || break
                    # Avoid double events
                    if ! echo "$event" | grep -e "on card" -e "on sink" -e "on server"; then
                        continue
                    fi
                fi
            } &>/dev/null
            output
        done
    }
}


function output() {
    if ! getCurSink; then
        echo "PulseAudio not running"
        return 1
    fi
    getCurVol "$curSink"
    getIsMuted "$curSink"

    # Fixed volume icons over max volume
    local iconsLen=${#ICONS_VOLUME[@]}
    if [ "$iconsLen" -ne 0 ]; then
        local volSplit=$((VOLUME_MAX / iconsLen))
        for i in $(seq 1 "$iconsLen"); do
            if [ $((i * volSplit)) -ge "$VOL_LEVEL" ]; then
                VOL_ICON="${ICONS_VOLUME[$((i-1))]}"
                break
            fi
        done
    else
        VOL_ICON=""
    fi

    getNickname "$curSink"

    # Showing the formatted message
    if [ "$isMuted" = "yes" ]; then
        # shellcheck disable=SC2034
        VOL_ICON=$ICON_MUTED
        echo "${COLOR_MUTED}$(eval echo "$FORMAT")${END_COLOR}"
    else
        eval echo "$FORMAT"
    fi
}


function usage() {
    echo "\
Usage: $0 [OPTION...] ACTION

Options:
  --autosync | --no-autosync
        Whether to maintain same volume for all programs.
        Default: $AUTOSYNC
  --color-muted <rrggbb>
        Color in which to format when muted.
        Default: ${COLOR_MUTED:4:-1}
  --notifications | --no-notifications
        Whether to show notifications when changing sinks.
        Default: $NOTIFICATIONS
  --osd | --no-osd
        Whether to display KDE's OSD message.
        Default: $OSD
  --icon-muted <icon>
        Icon to use when muted.
        Default: none
  --icon-sink <icon>
        Icon to use for sink.
        Default: none
  --format <string>
        Use a format string to control the output.
        Available variables:
        * \$VOL_ICON
        * \$VOL_LEVEL
        * \$ICON_SINK
        * \$SINK_NICKNAME
        Default: $FORMAT
  --icons-volume <icon>[,<icon>...]
        Icons for volume, from lower to higher.
        Default: none
  --volume-max <int>
        Maximum volume to which to allow increasing.
        Default: $VOLUME_MAX
  --volume-step <int>
        Step size when inc/decrementing volume.
        Default: $VOLUME_STEP
  --sink-blacklist <name>[,<name>...]
        Sinks to ignore when switching.
        Default: none
  --sink-nicknames-from <prop>
        pactl property to use for sink names, unless overriden by
        --sink-nickname. Its possible values are listed under the 'Properties'
        key in the output of \`pactl list sinks\`
        Default: none
  --sink-nickname <name>:<nick>
        Nickname to assign to given sink name, taking priority over
        --sink-nicknames-from. May be given multiple times, and 'name' is
        exactly as listed in the output of \`pactl list sinks short | cut -f2\`.
        Note that you can also specify a port name for the sink with
        \`<name>/<port>\`.
        Default: none

Actions:
  help              display this message and exit
  output            print the PulseAudio status once
  listen            listen for changes in PulseAudio to automatically update
                    this script's output
  up, down          increase or decrease the default sink's volume
  mute, unmute      mute or unmute the default sink's audio
  togmute           switch between muted and unmuted
  next-sink         switch to the next available sink
  sync              synchronize all the output streams volume to be the same as
                    the current sink's volume

Author:
    Mario Ortiz Manero
More info on GitHub:
    https://github.com/marioortizmanero/polybar-pulseaudio-control"
}

while [[ "$1" = --* ]]; do
    unset arg
    unset val
    if [[ "$1" = *=* ]]; then
        arg="${1//=*/}"
        val="${1//*=/}"
        shift
    else
        arg="$1"
        # Support space-separated values, but also value-less flags
        if [[ "$2" != --* ]]; then
            val="$2"
            shift
        fi
        shift
    fi

    case "$arg" in
        --autosync)
            AUTOSYNC=yes
            ;;
        --no-autosync)
            AUTOSYNC=no
            ;;
        --color-muted|--colour-muted)
            COLOR_MUTED="%{F#$val}"
            ;;
        --notifications)
            NOTIFICATIONS=yes
            ;;
        --no-notifications)
            NOTIFICATIONS=no
            ;;
        --osd)
            OSD=yes
            ;;
        --no-osd)
            OSD=no
            ;;
        --icon-muted)
            ICON_MUTED="$val"
            ;;
        --icon-sink)
            # shellcheck disable=SC2034
            ICON_SINK="$val"
            ;;
        --icons-volume)
            IFS=, read -r -a ICONS_VOLUME <<< "$val"
            ;;
        --volume-max)
            VOLUME_MAX="$val"
            ;;
        --volume-step)
            VOLUME_STEP="$val"
            ;;
        --sink-blacklist)
            IFS=, read -r -a SINK_BLACKLIST <<< "$val"
            ;;
        --sink-nicknames-from)
            SINK_NICKNAMES_PROP="$val"
            ;;
        --sink-nickname)
            SINK_NICKNAMES["${val//:*/}"]="${val//*:}"
            ;;
        --format)
	    FORMAT="$val"
            ;;
        *)
            echo "Unrecognised option: $arg" >&2
            exit 1
            ;;
    esac
done

case "$1" in
    up)
        volUp
        ;;
    down)
        volDown
        ;;
    togmute)
        volMute toggle
        ;;
    mute)
        volMute mute
        ;;
    unmute)
        volMute unmute
        ;;
    sync)
        volSync
        ;;
    listen)
        listen
        ;;
    next-sink)
        nextSink
        ;;
    output)
        output
        ;;
    help)
        usage
        ;;
    *)
        echo "Unrecognised action: $1" >&2
        exit 1
        ;;
esac
```

## Terminal
Existen varios emuladores de terminal, en mi caso hago uso de `gnome-terminal`.

### Archivo .zshrc
Este es el archivo de configuración que dispone de `temas`, `funciones`, `alias` y `plugins` para que el uso en consola sea mucho más fluido y cómodo.

Está situado en el directorio `~`:

```bash
# Enable Powerlevel10k instant prompt. Should stay close to the top of ~/.zshrc.
# Initialization code that may require console input (password prompts, [y/n]
# confirmations, etc.) must go above this block; everything else may go below.
if [[ -r "${XDG_CACHE_HOME:-$HOME/.cache}/p10k-instant-prompt-${(%):-%n}.zsh" ]]; then
  source "${XDG_CACHE_HOME:-$HOME/.cache}/p10k-instant-prompt-${(%):-%n}.zsh"
fi

# Set up the prompt

autoload -Uz promptinit
promptinit
prompt adam1

setopt histignorealldups sharehistory

# Use emacs keybindings even if our EDITOR is set to vi
bindkey -e

# Keep 1000 lines of history within the shell and save it to ~/.zsh_history:
HISTSIZE=1000
SAVEHIST=1000
HISTFILE=~/.zsh_history

# Use modern completion system
autoload -Uz compinit
compinit

zstyle ':completion:*' auto-description 'specify: %d'
zstyle ':completion:*' completer _expand _complete _correct _approximate
zstyle ':completion:*' format 'Completing %d'
zstyle ':completion:*' group-name ''
zstyle ':completion:*' menu select=2
eval "$(dircolors -b)"
zstyle ':completion:*:default' list-colors ${(s.:.)LS_COLORS}
zstyle ':completion:*' list-colors ''
zstyle ':completion:*' list-prompt %SAt %p: Hit TAB for more, or the character to insert%s
zstyle ':completion:*' matcher-list '' 'm:{a-z}={A-Z}' 'm:{a-zA-Z}={A-Za-z}' 'r:|[._-]=* r:|=* l:|=*'
zstyle ':completion:*' menu select=long
zstyle ':completion:*' select-prompt %SScrolling active: current selection at %p%s
zstyle ':completion:*' use-compctl false
zstyle ':completion:*' verbose true

zstyle ':completion:*:*:kill:*:processes' list-colors '=(#b) #([0-9]#)*=0=01;31'
zstyle ':completion:*:kill:*' command 'ps -u $USER -o pid,%cpu,tty,cputime,cmd'
source ~/powerlevel10k/powerlevel10k.zsh-theme

# To customize prompt, run `p10k configure` or edit ~/.p10k.zsh.
[[ ! -f ~/.p10k.zsh ]] || source ~/.p10k.zsh

# ==================================
# 	MANUAL CONFIGURATION
# ==================================

# === MAN COLORS ===
function man() {
    env \
    LESS_TERMCAP_mb=$'\e[01;31m' \
    LESS_TERMCAP_md=$'\e[01;31m' \
    LESS_TERMCAP_me=$'\e[0m' \
    LESS_TERMCAP_se=$'\e[0m' \
    LESS_TERMCAP_so=$'\e[01;44;33m' \
    LESS_TERMCAP_ue=$'\e[0m' \
    LESS_TERMCAP_us=$'\e[01;32m' \
    man "$@"
}

# === ALIAS ===

# batcat
alias cat="/usr/bin/batcat --paging never"
alias catm="/usr/bin/batcat"
alias catn="/usr/bin/cat"

# lsd 
alias ls="/usr/bin/lsd --group-dirs=first"
alias la="/usr/bin/lsd -a --group-dirs=first"
alias ll="/usr/bin/lsd -lh --group-dirs=first"
alias lla="/usr/bin/lsd -lah --group-dirs=first"

# grep
alias grep="/usr/bin/grep --color=auto"

# === PLUGINS ===
source /usr/share/zsh-autosuggestions/zsh-autosuggestions.zsh
source /usr/share/zsh-syntax-highlighting/zsh-syntax-highlighting.zsh
source /usr/share/zsh-sudo/sudo.plugin.zsh
```