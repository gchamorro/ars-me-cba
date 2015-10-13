# YUMI efi v1.0

Instalar con YUMI las versiones tanto de 32 bits como de 64 bits efi de las herramientas que necesitamos, booteables en ambas arquitecturas en un único pendrive.

---

# Contenido

- [Obtener el software necesario](#obtener-el-software-necesario)

- [Instalación](#instalacin)

    - [Rescatux](#rescatux)
    
    - [Huayra](#huayra) 

- ["Hackeo" para permitir el booteo uefi](#hackeo-para-permitir-el-booteo-efi)

    - [Editar el archivo grub.cfg raiz](#editar-el-archivo-grubcfg-raiz)
    
    - [Reemplazar los grub.cfg propios de cada herramienta](#reemplazar-los-grubcfg-propios-de-cada-herramienta)    

- [Testeo](#testeo)

- [Guía rápida, me sale todo al toque](#gua-rpida-me-sale-todo-al-toque)

- [Preguntas frecuentes](#preguntas-frecuentes)

- [Problemas conocidos](#problemas-conocidos)

---

## 1. Obtener el software necesario

* Bajar las imágenes iso de cada una de las herramientas, en este caso utilizamos solo las de 64 bits de [Clonezilla](http://www.clonezilla.org/downloads.php), [GParted](http://sourceforge.net/projects/gparted/files/gparted-live-stable/) y [Huayra GNU/Linux](http://huayra.conectarigualdad.gob.ar/iso-sistema#block-views-isos-menu-iso-ultima-estable), ya que se ejecutan en ambas arquitecturas. La imagen de [Rescatux](http://www.supergrubdisk.org/category/download/rescatuxdownloads/rescatux-stable/]) es híbrida en arquitectura.

* Bajar la última versión de [YUMI](http://www.pendrivelinux.com/yumi-multiboot-usb-creator/) para Windows (la versión para linux es problemática, no la recomiendo).

* Bajar el siguiente archivo [yumi_efi_raiz](/ars-me-cba/public/archivos/yumi_efi_raiz.zip).

* Bajar el [clonezilla.cfg para el clonezilla](/ars-me-cba/public/archivos/clonezilla.cfg).

* Bajar el [gparted.cfg para el gparted](/ars-me-cba/public/archivos/gparted.cfg).

---

## 2. Instalación

Instalar normalmente las imágenes iso en YUMI tanto de Clonezilla como de GParted.

### Rescatux

Rescatux se compone de una iso dentro de otra. Extraer la imagen iso dentro de la imagen iso. Valga el trabalenguas.

En nuestro ejemplo nos bajamos la iso *rescatux_cdrom_usb_hybrid_i386_amd64-486_0.30.2_sg2d.iso*, la cual abrimos con algún software extractor como winrar o file roller (linux), y extraemos la imagen interna *rescatux_cdrom_usb_hybrid_i386_amd64-486_0.30.2.iso*.

Esta última imagen extraída es la que instalaremos con el YUMI.

### Huayra

La opción a elegir en YUMI al instalar es _Try unlisted ISO (SYSLINUX)_

---

## 3. "Hackeo" para permitir el booteo efi

Extraemos el contenido del zip que bajamos en el punto 1 [yumi_efi_raiz.zip](/ars-me-cba/public/archivos/yumi_efi_raiz.zip). Se extraerán entonces las carpetas _EFI_ y _boot_. **Atención:** si al extraer se crea una carpeta con el nombre del zip, buscar dentro las carpetas mencionadas. **EFI y boot deben ser copiadas en la raiz del usb.**


La carpeta _boot_ contiene la subcarpeta _grub_, y dentro de ésta el archivo _grub.cfg_, al que llamaremos _raiz_.

### Editar el archivo `grub.cfg` raiz

Este archivo será nuestro menú principal EFI, creando las entradas apuntando a los archivos _grub.cfg_ específicos de cada instalación. 
Cada vez que realicemos una instalación nueva debemos agregar la entrada correspondiente, prestando especial anteción a la ruta donde se encuentra el archivo al que se hace referencia.

Ejemplo:

_Contenido del archivo [raiz_del_usb]/EFI/boot/grub.cfg_

{% highlight bash %}  
    #para que bootee el clonezilla
    menuentry "Clonezilla" {
        configfile /multiboot/clonezilla-live-2.4.2-32-amd64/EFI/boot/clonezilla.cfg
    }

    #para que bootee el gparted
    menuentry "Gparted" {
        configfile /multiboot/gparted-live-0.23.0-1-amd64/EFI/boot/gparted.cfg
    }
{% endhighlight %}

### Reemplazar los grub.cfg propios de cada herramienta

#### Clonezilla

Copiar para el clonezilla el archivo [clonezilla.cfg](/ars-me-cba/public/archivos/clonezilla.cfg) en el directorio `/multiboot/[directorio_clonezilla]/EFI/boot/`


#### Gparted

Copiar para el gparted el archivo [gparted.cfg](/ars-me-cba/public/archivos/gparted.cfg) en el directorio `/multiboot/[directorio_gparted]/EFI/boot/`

**Cuando el grub.cfg propio de la herramienta no existe (o no se puede hacer funcionar)**, deberemos crear una entrada específica. Aquí están los *menuentry* correspondientes a Rescatux y Huayra:


{% highlight bash %}  
    menuentry "Rescatux"{
        search --set -f /multiboot/rescatux_cdrom_usb_hybrid_i386_amd64-486_0.30.2/live/vmlinuz
        linux /multiboot/rescatux_cdrom_usb_hybrid_i386_amd64-486_0.30.2/live/vmlinuz boot=live ignore_bootid live-media-path=/multiboot/rescatux_cdrom_usb_hybrid_i386_amd64-486_0.30.2/live config   quiet
        initrd /multiboot/rescatux_cdrom_usb_hybrid_i386_amd64-486_0.30.2/live/initrd.img
    }

    menuentry "Huayra 3.1 amd64" {
        linux /multiboot/huayra-amd64-3.1-efi/live/vmlinuz noprompt live-media-path=/multiboot/huayra-amd64-3.1-efi/live  boot=live config file=/preseed.cfg language=es locales=es_AR.UTF-8 country=AR keyboard-layouts=es
        initrd /multiboot/huayra-amd64-3.1-efi/live/initrd.img
    }
{% endhighlight %}

#### El contenido del archivo **_/EFI/boot/grub.cfg_** queda entonces así:


{% highlight bash %}  
    menuentry "Clonezilla" {
        configfile /multiboot/clonezilla-live-2.4.2-32-amd64/EFI/boot/clonezilla.cfg
    }

    menuentry "Gparted" {
        configfile /multiboot/gparted-live-0.23.0-1-amd64/EFI/boot/gparted.cfg
    }

    menuentry "Rescatux"{
        search --set -f /multiboot/rescatux_cdrom_usb_hybrid_i386_amd64-486_0.30.2/live/vmlinuz
        linux /multiboot/rescatux_cdrom_usb_hybrid_i386_amd64-486_0.30.2/live/vmlinuz boot=live ignore_bootid live-media-path=/multiboot/rescatux_cdrom_usb_hybrid_i386_amd64-486_0.30.2/live config   quiet
        initrd /multiboot/rescatux_cdrom_usb_hybrid_i386_amd64-486_0.30.2/live/initrd.img
    }

    menuentry "Huayra 3.1 amd64" {
        linux /multiboot/huayra-amd64-3.1-efi/live/vmlinuz noprompt live-media-path=/multiboot/huayra-amd64-3.1-efi/live  boot=live config file=/preseed.cfg language=es locales=es_AR.UTF-8 country=AR keyboard-layouts=es
        initrd /multiboot/huayra-amd64-3.1-efi/live/initrd.img
    }
{% endhighlight %}

---

## Testeo

Testeadas en las netbooks:

- EXO X352 (32 bits) 
- CORADIR CDR Class (64 bits uefi)

Testeadas las imágenes de:

* Huayra 3.1 amd64
* Gparted 0.23.0-1-amd64
* Clonezilla 2.4.2-32-amd64
* Rescatux i386_amd64-486_0.30.2

---

## Guía rápida, me sale todo al toque

1. Instalar con la última versión de **YUMI**, `clonezilla-live-2.4.2-32-amd64`,  `gparted-live-0.23.0-1-amd64`, `rescatux_cdrom_usb_hybrid_i386_amd64-486_0.30.2` (desempaquetado de la iso original), `huayra-amd64-3.1-efi`.

2. Copiar las carpetas `EFI` y `boot` provistas en [yumi_efi_raiz.zip](/ars-me-cba/public/archivos/yumi_efi_raiz.zip) en la raíz del usb.

3. Copiar para el clonezilla el archivo [clonezilla.cfg](/ars-me-cba/public/archivos/clonezilla.cfg) en el directorio `/multiboot/[directorio_clonezilla]/EFI/boot/` y el archivo [gparted.cfg](/ars-me-cba/public/archivos/gparted.cfg) en el directorio `/multiboot/[directorio_gparted]/EFI/boot/`

4. Listo el pollo.

---

## Preguntas frecuentes

`...` 

---

## Problemas conocidos

`...` 

---
