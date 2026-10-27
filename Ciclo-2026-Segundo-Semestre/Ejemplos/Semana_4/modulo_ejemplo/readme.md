# Módulo de ejemplo

Esta carpeta contiene un módulo de kernel mínimo y el `Makefile` necesario
para compilarlo contra el kernel activo.

## Preparar el equipo

En Ubuntu o Debian instala las dependencias:

```bash
sudo apt update
sudo apt install -y build-essential linux-headers-$(uname -r)
```

## Compilar y probar

Ejecuta estos comandos dentro de esta carpeta:

```bash
make
sudo insmod modulo_ejemplo.ko
lsmod | grep modulo_ejemplo
sudo dmesg | tail -n 20
```

El módulo escribe mensajes al cargarse. Para retirarlo:

```bash
sudo rmmod modulo_ejemplo
sudo dmesg | tail -n 20
```

Limpia los archivos generados con `make clean`.
