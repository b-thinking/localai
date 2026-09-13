# Instalación
## Prerequisitos
### Podman: Instalación
Podman es el motor de contenedores oficial para RedHat 9 y 10 (y por consiguiente Oracle Linux 9 y 10). Ha sustituido al motor de Docker que se usaba en RedHat 7 y 8. 

Dependiendo de la distribución utilizada puede venir instalado, o deberás instalarlo con 

- Instalación completa: Podman + Buildah + Skopeo. Recomendada si vas a construir y publicar contenedores desde tu máquina
    ```shell
    sudo dnf module install container-tools
    ```
- Instalación mínima: Si sólo vas a ejecutar contenedores ya creados
    ```shell
    sudo dnf install podman
    ```

NOTA: Para otras distribuciones de Linux (soporta todas o casi todas) mira la página [Podman Installation](https://podman.io/docs/installation). Ten en cuenta

- Se requiere una versión de podman superior a 5.x para el soporte del acceso a GPU desde los contenedores.
- La configuración incluida aquí usa la aproximación rootless de systemd. Si tu distribución NO es systemd (seguro que sabes de Linux más que yo) deberás tenerlo en cuenta

### GPU

Para el uso de la GPU de NVIDIA en entorno Linux se decide instalar lo mínimo a nivel de host, dejando la instalación de CUDA como parte del contenedor. La razón de esta aproximación es que cada servicio probado dependendía de versiones de CUDA diferentes, y que además no eran parte de los repositorios básicos de Oracle Linux.

Por tanto, instalaremos los componentes mínimos, que son las siguientes:

#### [Driver con soporte a CUDA](https://docs.nvidia.com/datacenter/tesla/driver-installation-guide/latest/red-hat-enterprise-linux.html). IMPORTANTE: No instalamos CUDA completo, sólo el driver.
1. Instalamos el soporte de driver dinámico para el Kernel de NVIDIA (nvidia-dkms). Los siguientes son las instrucciones para Oracle Linux. Para tu distribución, busca NVIDIA OPEN DKMS.
    - Instalamos el paquete open dkms
        ```shell
        sudo dnf install -y kmod-nvidia-latest-dkms
        ```
    - Verificamos que se haya instalado correctamente. NOTA: Debe aparecer una línea por cada kernel que tengas configurado en tu sistema
        ```shell
        dkms status
        # Output
        # nvidia/610.57.04, 6.12.0-211.50.1.el10_2.x86_64, x86_64: installed
        ```
    - Opcional pero recomendado: Reiniciamos (`sudo reboot`) el sistema y tras arranque verificamos los módulos cargados
        ```shell
        lsmod | grep nvidia
        # **nvidia**_uvm           2449408  0
        # **nvidia**_drm            151552  4
        # **nvidia**_modeset       1818624  1 **nvidia**_drm
        # **nvidia**              105889792  2 **nvidia**_uvm,**nvidia**_modeset
        # drm_ttm_helper         16384  2 **nvidia**_drm
        # video                  81920  1 **nvidia**_modeset
        ```
2. Instalamos el driver de NVIDIA.
    ```shell
    sudo dnf install -y nvidia-driver-cuda
    ```
3. Opcional pero recomendado: Activamos el servicio de persistencia del driver en memoria. NOTA: Aumenta el consumo pero reduce la latencia.
    ```shell
    sudo systemctl enable --now nvidia-persistenced
    sudo systemctl start nvidia-persistenced
    sudo systemctl status nvidia-persistenced
    ```
4. Verificamos que el driver esté cargado
    ```shell
    nvidia-smi
    ```
#### Soporte para contendores: [NVIDIA Container Toolkit](https://github.com/NVIDIA/nvidia-container-toolkit)
1. Instalar el **tookit**
    ```shell
    # Install
    sudo dnf install -y \
        nvidia-container-toolkit \
        nvidia-container-toolkit-base
    # Check
    which nvidia-ctk
    nvidia-ctk --version
    ```
2. Generar el **interfaz** de contenedores. NOTA: Esto es muy dependendiente del sistema de contenedores que estés usando (podman, docker, rancher) y si lo estás usando rootless o no. Las instrucciones que incluyo aquí son las que he probado con podman rootless. Si tu caso es diferente, mira la documentación, que es bastante extensa y completa.
    - Generar el archivo de configuración. Durante la generación se producirán warnings sobre servicios que no quedan disponibles para contenedores. En nuestro caso sólo queremos usar el interfaz para IA, y sólo tengo una GPU. 
        ```shell
        sudo nvidia-ctk cdi generate --output=/etc/cdi/nvidia.yaml
        # Discared warnings
        # 1) Wayland / EGL warnings (server headless - no desktop)
        # 2) X11 warnings (server headless - no desktop)
        # 3) Vulkan warnings (server headless - no desktop - no gaming)
        # 4) Fabric manager: (Multiple / Datacenter fabrics)
        # 5) MPS service (Multiple HPC)
        # Check
        cat /etc/cdi/nvidia.yaml
        ```
    - Verificar que dispositivos se han generado (que son los que usaremos luego en los contenedores)
        ```shell
        sudo nvidia-ctk cdi list
        # INFO[0000] Found 3 CDI devices                          
        # nvidia.com/gpu=0
        # nvidia.com/gpu=GPU-a45fefa4-2cad-3003-b6a6-1464ac1a491a
        # nvidia.com/gpu=all
        ```
    - Autorizar el uso de los dispositivos GPU a los contenedores en SELinux
        ```shell
        sudo setsebool -P container_use_devices 1
        ```
    - Probar con un **contenedor básico CUDA** que tiene acceso a la tarjeta
        ```shell
        podman run --rm \
            --device nvidia.com/gpu=all \
            docker.io/nvidia/cuda:13.0.1-base-ubi9 \
            nvidia-smi
        ```
    - **ALTERNATIVA**: Si te sigue dando problemas de seguridad, **Y SÓLO PARA PROBAR**, se puede arrancar el contenedor sin etiquetado SELinux
        ```shell
        podman run --rm \
            --security-opt=label=disable \
            --device nvidia.com/gpu=all \
            docker.io/nvidia/cuda:13.0.1-base-ubi9 \
            nvidia-smi
        ```

## Podman: Configuración
### Reglas generales
- **Modo Rootless**: Los contenedores se desplegarán como servicios `systemd`de tu usuario. Sólo se dará acceso root cuando sea inevitable. Además se mantendrá activo el soporte de seguridad SELinux.
- **Registro**: Siempre que sea posible, se usará el registro de GitHub (`ghcr.io`). Como alternativa, tenemos el de docker `docker.io`o bien el registro privado que se despliegue a nivel de empresa
- **Versiones del contenedor**: Usaremos versiones fijas, evitando la etiqueta `latest`para tener un entorno reproducible. Es importante que estos productos despligan versiones muy habitualmente, por lo que se recomienda revisar las versiones desplegadas y actualizarlas. Para ello se puede utilizar este propio repositorio etiquetando el subdirectorio de contenedores con la versión probada en cada momento.
- **Comunicación entre contenedores**: La comunicación entre los contenedores se realizará en base al nombre de cada uno, usando el servicio DNS interno de Podman. Por tanto, es importante definir correctamente el atributo `ContainerName`de cada contenedor
### Configuración
- **Modo Rootless**: Al ser los contenedores servicios `systemd`de tu usuario, su configuración está ubicada en `$HOME/.config`. Específicamente, los [contenedores](./containers/systemd/) deberán ubicarse en `$HOME/.config/containers.systemd/`y la configuración de las [variables de entorno](./environment.d/) deberán ubicarse en `$HOME/.config/environment.d`.
- **Variables de entorno**: Se ha definido un archivo de configuración para las variables de entorno comunes a todo el proyecto en [ai.conf](./environment.d/ai.conf). NOTA IMPORTANTE: Usar este archivo es neceario al operar en modo persistente (Linger) puesto que en este caso los scripts habituales (.profile, .bashrc, ...) no se ejecutan al iniciar los contenedor.
- **Linger**: Por omisión, al ser servicios de usuario, estos no arrancan hasta que el usuario se conecta. Si deseas que estos servicios estén disponibles siempre, debes activar el linger para ese usuairo con
    ```shell
    sudo loginctl enable-linger NOMBRE_DE_USUARIO
    ```
### Uso básico
- **Despliegue**: Los contenedores en modo quadlet se despliegan regenerando su configuración cada vez que se modifica un archivo de contenedor u otra configuración (red, entorno). Para ello, se ejecuta (a nivel de usuario, sin `sudo`)
    ```bash
    systemctl --user daemon-reload
    ```
- **Errores de sintaxis**: Normalmente, el daemon-reload no indica los errores de sintaxis (simplemente no genera el servicio). En caso de que se produzcan, se debe corregir el archivo de configuración de contenedor correspondiente. Para identificar los errores de sintaxis siempre que se añada o modifique un archivo de contenedor.
    - **Dryrun**: Generar en vacío todos los contenedores. Si hay errores, lo indica
        ```shell
        # Dry-run generation of service files
        /usr/lib/systemd/system-generators/podman-system-generator --user --dryrun
        ```
    - **Verificar** un servicio concreto (ideal para scripts DevOps) 
        ```shell
        # Verify a specific service. If is not generated or error, return error. Else nothing
        systemd-analyze --user --generators=true verify your_container_file_name.service
        ```
- **Gestión de los servicios**: La gestión es la habitual de systemd, indicando el modo `--user`. IMPORTANTE: Los servicios los desplegamos con dependencias entre ellos. Tengase en cuanta que parar un servicio requerido por otro, finaliza ambos.
    - **Iniciar**: `systemctl --user start your_container_file_name.service`
    - **Parar**: `systemctl --user stop your_container_file_name.service`
    - **Reiniciar**: `systemctl --user restart your_container_file_name.service`
    - **Estado**: `systemctl --user status your_container_file_name.service`

## Red de contenedores
## Motor de inferencia: Ollama
## Framework AI: Open WebUI
## Base de datos vectorial para RAG: Chroma
## Servicio de búsqueda: SearxNG
# Ejemplos de uso