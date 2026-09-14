# Instalación
## Prerequisitos
### Podman: Instalación

Podman tiene instación el Mac de dos maneras: Mediante un instalador nativo (archivo `.dmg`) o mediante `brew``

Además, se debe tener en cuenta que está disponible en dos sabores: Línea de comandos únicamente (Podman CLI) y entorno de escritorio (Podman desktop).

El entorno de escritorio es más código para administrar, e incluye todas las funcionalidades de la línea de comandos.

Para este laboratorio está instalado podman CLI en la versión 6.1

```shell
# Only Podman-CLI
brew install podman
# Alternative: Install desktop
# brew install --cask podman-desktop
```

### GPU Metal

PENDIENTE

## Podman: Configuración
### Reglas generales
Asumismos las mismas reglas generales definidas para Linux en [Reglas generales - Linux](README.linux.es.md#reglas-generales)
### Configuración
- **Máquina virtual**: La consideración más importante en MacOS es que podman no es nativo y requiere la ejecución de una máquina virtual linux dentro de MacOS. Se pueden tener una o varias máquinas virtuales, y pueden pararse y arrancarse a demanda. 
    - Inicializar la máquina: `podman machine init [NAME]` crea una máquina virtual. Si no se indica el nombre por defecto es`podman-machine-default`. Como todo modelo de máquinas virtuales, se puede configurar la memoria, la CPU y modo de red. Para ver todos los parámetros `podman machine init --help`. Los parámetros por omisión son 6 CPUs, 100 GB de disco, 2GB de memoria. Estos valores pueden cambiar sin recrear la máquina con `podman machine set` 
    - Lista de máquinas virtuales: `podman machine list` 
    - Arrancar una máquina virtual: `podman machine start [NAME]`
    - Parar una máquina virtual: `podman machine stop [NAME]`
    - Borrar una máquina virtual: `podman machine rm [NAME]
    - Cambiar una máquina virtual (sin necesidad de recrearla): `podman machine set`
- **Virtualizar ruta `/vm`**. En el Mac no se pueden (ni deben) crear carpetas en el raiz. Para compatibilidad y coherencia con la instalación en Linux, los volumenes se crean en `/vm/ai`, por lo que debemos virtualizar esta ruta. 
    - Editamos archivo de configuración `sudo nano /etc/synthetic.conf`
    - El contenido debe estar separado por único caracter de tabulación, y no contener la barra de inicio
        ```conf
        vm  Users/jmalbarran/Virtual Machines.localized
        ```
    - Reiniciar el ordenador y verificar `ls -la /vm`
    - NOTA: Ayuda adicional: `man synthetic.conf`
- **Configuración Quadlet**: La maquina virtual en si misma es una systemd (se puede elegir imagen) y se puede acceder a la misma con el comando `podman machine ssh`. Para facilitar la edición, he enlazado el directorio de la máquina virtual con el correspondiente del host.
    ```shell
    # Access to VM
    podman machine ssh
    # Inside VM
    ln -s /Users/jmalbarran/.config/containers/systemd/ ~/.config/containers/systemd
    ```

### Uso básico
El uso básico debemos distinguir el que se ejecuta dentro de la máquina virtual (mandatos `systemctl --user`) y los mandatos propios de podman.

Los mandatos systemctl se ejecutarán en la máquina virtual usando el ssh. Por ejemplo:
```shell
podman machine ssh 'systemctl --user daemon-reload'
```

Los mandatos básicos son los mismos que los de [Uso básico - Linux](README.linux.es.md#uso-básico) pero prefijanddo con `podman machine ssh``


