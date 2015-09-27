---
layout: instructivo
title: Yumi EFI
---

# Iumi + uendous 8 (uefi) + rescatus + uaira

Instalar con yumi las versiones tanto de 32 bits como de 64 bits uefi de las herramientas que necesitamos, booteables en ambas arquitecturas en un único pendrive.

Testeadas en las netbooks EXO X352 (32 bits) y CORADIR CDR Class (64 bits uefi):

* Huayra 3.1 amd64
* Gparted 0.23.0-1-amd64
* Clonezilla 2.4.2-32-amd64
* Rescatux i386_amd64-486_0.30.2

---

## 1. Obtener el software necesario

* Bajar las imágenes iso de cada una de las herramientas, en este caso utilizamos solo las de 64 bits de [Clonezilla](http://www.clonezilla.org/downloads.php), [GParted](http://sourceforge.net/projects/gparted/files/gparted-live-stable/) y [Huayra GNU/Linux](http://huayra.conectarigualdad.gob.ar/iso-sistema#block-views-isos-menu-iso-ultima-estable), ya que se ejecutan en ambas arquitecturas. La imagen de [Rescatux](http://www.supergrubdisk.org/category/download/rescatuxdownloads/rescatux-stable/]) es híbrida en arquitectura.

* Bajar la última versión de [YUMI](http://www.pendrivelinux.com/yumi-multiboot-usb-creator/) para Windows (La versión para linux es problemática, no la recomiendo).

* Bajar el siguiente archivo [yumi_raiz_efi](https://drive.google.com/file/d/0B_pDv9cL1hOLRWJKOWlpaUFmb3M/view?usp=sharing).

---

## 2. Instalación

Instalar normalmente las imágenes iso en YUMI tanto de Clonezilla como de GParted.

### Rescatux

Rescatux se compone de una iso dentro de otra. Extraer la imagen iso dentro de la imagen iso. Valga el trabalenguas.

En nuestro ejemplo nos bajamos la iso *rescatux_cdrom_usb_hybrid_i386_amd64-486_0.30.2_sg2d.iso*, la cual abrimos con algún software extractor como winrar o file roller (linux), y extraemos la imagen interna *rescatux_cdrom_usb_hybrid_i386_amd64-486_0.30.2.iso*.

Esta última imagen extraída es la que instalaremos con el YUMI.

### Huayra

La opción a elegir en YUMI al instalar es *Try unlisted ISO (SYSLINUX)*

### Otros...

---

## 3. "Hackeo" para permitir el booteo uefi

Extraemos el contenido del zip que bajamos en el punto 1 con el nombre *yumi_efi_raiz.zip*. Se extraerán entonces las carpetas _EFI_ y _boot_. **Atención:** al extraer se crea una carpeta con el nombre del zip, y dentro las carpetas mencionadas. **EFI y boot deben ser copiadas en la raiz del usb.**


La carpeta _boot_ contiene la subcarpeta _grub_, y dentro de ésta el archivo _grub.cfg_, al que llamaremos _raiz_.

### Editar el archivo grub.cfg raiz:

Este archivo será nuestro menú principal EFI, creando las entradas apuntando a los archivos _grub.cfg_ específicos de cada instalación. 
Cada vez que realicemos una instalación nueva debemos agregar la entrada correspondiente, prestando especial anteción a la ruta donde se encuentra el archivo al que se hace referencia.

Ejemplo:

_Contenido del archivo [raiz_del_usb]/EFI/boot/grub.cfg_

	#para que bootee el clonezilla
	menuentry "Clonezilla" {
		configfile /multiboot/clonezilla-live-2.4.2-32-amd64/EFI/boot/grub.cfg
	}

	#para que bootee el gparted
	menuentry "Gparted" {
		configfile /multiboot/gparted-live-0.23.0-1-amd64/EFI/boot/grub.cfg
	}

	...


**Cuando el grub.cfg propio de la herramienta no existe**, deberemos crear una entrada específica. Aquí están los *menuentry* correspondientes a Rescatux y Huayra:

	#para que bootee el rescatux
	menuentry "Rescatux"{
		search --set -f /multiboot/rescatux_cdrom_usb_hybrid_i386_amd64-486_0.30.2/live/vmlinuz
		linux /multiboot/rescatux_cdrom_usb_hybrid_i386_amd64-486_0.30.2/live/vmlinuz boot=live ignore_bootid live-media-path=/multiboot/rescatux_cdrom_usb_hybrid_i386_amd64-486_0.30.2/live config   quiet
		initrd /multiboot/rescatux_cdrom_usb_hybrid_i386_amd64-486_0.30.2/live/initrd.img
}

	#para que bootee el huayra
	menuentry "Huayra 3.1 amd64" {
		linux /multiboot/huayra-amd64-3.1-efi/live/vmlinuz noprompt live-media-path=/multiboot/huayra-amd64-3.1-efi/live  boot=live config file=/preseed.cfg language=es locales=es_AR.UTF-8 country=AR keyboard-layouts=es
		initrd /multiboot/huayra-amd64-3.1-efi/live/initrd.img
	}

	...

El contenido del archivo **_/EFI/boot/grub.cfg_** queda entonces así:

	menuentry "Clonezilla" {
		configfile /multiboot/clonezilla-live-2.4.2-32-amd64/EFI/boot/grub.cfg
	}

	menuentry "Gparted" {
		configfile /multiboot/gparted-live-0.23.0-1-amd64/EFI/boot/grub.cfg
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


#### NOTA: Utilizando las versiones descritas en este instructivo, el archivo _grub.cfg_ ya tiene las entradas correspondientes, por lo que no sería necesario editarlo. Con solo instalar las imágenes y extraer las carpetas EFI y boot en la raiz, el pendrive estaría listo para usar.

---

