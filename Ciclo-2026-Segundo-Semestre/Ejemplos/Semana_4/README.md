# Semana 4: módulo de kernel

Esta semana contiene dos archivos relacionados con información del sistema:

- `sysinfo.c` es una versión más completa que expone información mediante
  `/proc/sysinfo`.
- `modulo_ejemplo/modulo_ejemplo.c` es un módulo pequeño que registra mensajes
  al cargarlo y retirarlo.

La compilación de módulos de kernel debe hacerse sobre una máquina Linux. El
módulo debe coincidir con la versión del kernel en ejecución.

## Requisitos

En Ubuntu o Debian instala las herramientas de compilación y los headers del
kernel:

```bash
sudo apt update
sudo apt install -y build-essential linux-headers-$(uname -r)
```

## Compilar y cargar el módulo de ejemplo

Entra en la carpeta del módulo y compílalo:

```bash
cd modulo_ejemplo
make
```

El resultado es `modulo_ejemplo.ko`. Cárgalo y revisa los mensajes del kernel:

```bash
sudo insmod modulo_ejemplo.ko
lsmod | grep modulo_ejemplo
sudo dmesg | tail -n 20
```

Cuando termines, retíralo:

```bash
sudo rmmod modulo_ejemplo
sudo dmesg | tail -n 20
```

Para borrar los archivos generados:

```bash
make clean
```

## Sobre `sysinfo.c`

`sysinfo.c` contiene el código de otro módulo que recorre procesos de
`containerd-shim` y publica información en `/proc/sysinfo`. Su compilación
requiere un `Makefile` equivalente al de `modulo_ejemplo`. Antes de cargarlo,
revisa el código para adaptarlo a la versión del kernel de la máquina.

Los módulos se ejecutan con privilegios del kernel. Úsalos en una máquina de
pruebas y no en un servidor que contenga información importante.
