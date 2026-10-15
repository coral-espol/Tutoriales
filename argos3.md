# Instalación de ARGoS3

[ARGoS3](https://www.argos-sim.info) es un simulador de enjambres de robots. Esta guía indica qué versión instalar según tu sistema operativo y cómo hacerlo, para que todos en el laboratorio trabajemos con la misma versión.

## Qué versión usar

La última versión publicada de ARGoS3 es la **3.0.0-beta59**. Los paquetes `.deb` oficiales solo existen hasta Ubuntu 22.04; en Ubuntu 24.04 hay que compilar desde el código fuente.

| Sistema operativo | Método recomendado | Versión que queda instalada |
| --- | --- | --- |
| **Ubuntu 24.04** | [Compilar desde el código fuente](#opción-a-compilar-desde-el-código-fuente-recomendada) | 3.0.0-beta59 + correcciones posteriores (rama `master`) |
| **Ubuntu 22.04** | Compilar desde el código fuente, o el [paquete `.deb`](#opción-b-paquete-deb-solo-ubuntu-2004-y-2204) para 22.04 | 3.0.0-beta59 |
| **Ubuntu 20.04** | Compilar desde el código fuente, o el [paquete `.deb`](#opción-b-paquete-deb-solo-ubuntu-2004-y-2204) para 20.04 | 3.0.0-beta59 |
| Ubuntu 18.04 o anterior | No recomendado (sin soporte de Ubuntu). Actualiza a 22.04 o 24.04 | — |
| macOS | [Homebrew](https://www.argos-sim.info/core.php): `brew tap ilpincy/argos3` y `brew install bash-completion qt argos3` | 3.0.0-beta59 |
| Windows | No soportado. Usa Ubuntu en arranque dual o en máquina virtual | — |

**Recomendación para el laboratorio:** compila desde el código fuente (opción A) en cualquier versión de Ubuntu. Así todos tienen las mismas funciones, sin importar si usan 20.04, 22.04 o 24.04. La rama `master` incluye mejoras posteriores a la beta59 (por ejemplo, métodos para fijar la posición y orientación inicial de un robot) que el `.deb` no trae: código que las use no compilará con el `.deb`.

Para saber qué Ubuntu tienes: `lsb_release -d`.

## Opción A: compilar desde el código fuente (recomendada)

Funciona en Ubuntu 20.04, 22.04 y 24.04. Tarda entre 5 y 15 minutos.

**1. Dependencias:**

```bash
sudo apt update
sudo apt install git build-essential cmake \
  libfreeimage-dev libfreeimageplus-dev \
  qtbase5-dev freeglut3-dev libxi-dev libxmu-dev \
  liblua5.3-dev lua5.3
```

> Guías antiguas piden `qt5-default`: ese paquete ya no existe desde Ubuntu 22.04. `qtbase5-dev` lo reemplaza.

**2. Descargar y compilar:**

```bash
cd ~
git clone https://github.com/ilpincy/argos3.git
cd argos3
mkdir build_simulator && cd build_simulator
cmake -DCMAKE_BUILD_TYPE=Release -DARGOS_DOCUMENTATION=OFF ../src
make -j$(nproc)
```

En la salida de `cmake` revisa que aparezcan `Found Qt5Widgets` y `Found Lua`; si falta Qt, ARGoS se compila **sin ventana gráfica**. Los `warning` de `make` son normales; solo importa que termine en `[100%]` sin `Error`.

**3. Instalar:**

```bash
sudo make install
sudo ldconfig
```

ARGoS queda en `/usr/local`. No borres la carpeta `~/argos3/build_simulator`: la necesitas para desinstalar.

## Opción B: paquete `.deb` (solo Ubuntu 20.04 y 22.04)

1. Entra a https://www.argos-sim.info/core.php y busca la fila **Binary Packages for Ubuntu** de **tu versión** (22.04 o 20.04).
2. Descarga `argos3_simulator-3.0.0-x86_64-beta59.deb`. **Ojo:** el archivo se llama igual para todas las versiones de Ubuntu; descárgalo de la fila correcta.
3. Instálalo desde la carpeta de descargas:

   ```bash
   cd ~/Descargas   # o ~/Downloads
   sudo apt install ./argos3_simulator-3.0.0-x86_64-beta59.deb
   ```

No uses este `.deb` en Ubuntu 24.04: está hecho para las librerías de 22.04. En 24.04 usa la opción A.

## Comprobar la instalación

```bash
argos3 --version
```

Debe mostrar `ARGOS_VERSION=3.0.0-beta59`. Luego prueba un experimento con ventana gráfica usando los ejemplos oficiales:

```bash
cd ~
git clone https://github.com/ilpincy/argos3-examples.git
cd argos3-examples
mkdir build && cd build
cmake -DCMAKE_BUILD_TYPE=Release ..
make -j$(nproc)
cd ..
argos3 -c experiments/diffusion_1.argos
```

Se abre la ventana de ARGoS con un foot-bot en una arena. Pulsa ▶ (Play) para que se mueva evitando obstáculos.

## Desinstalar o cambiar de método

No mezcles las dos opciones: antes de cambiar de método, desinstala la que tengas.

| Instalado con | Desinstalar |
| --- | --- |
| Opción A (código fuente) | `cd ~/argos3/build_simulator && sudo xargs rm -f < install_manifest.txt && sudo ldconfig` |
| Opción B (`.deb`) | Busca el nombre del paquete con `dpkg -l \| grep -i argos` y luego `sudo apt remove <nombre>` |

## Problemas frecuentes

| Problema | Solución |
| --- | --- |
| `argos3: command not found` | No se completó `sudo make install`. Vuelve a ejecutarlo en `~/argos3/build_simulator`. |
| `error while loading shared libraries: libargos3core_simulator.so` | Ejecuta `sudo ldconfig`. |
| ARGoS corre pero no abre la ventana gráfica | Faltaban las librerías de Qt al compilar. Instala las dependencias del paso 1, borra `build_simulator` y vuelve a compilar. |
| `Unable to locate package qt5-default` | Ese paquete no existe desde Ubuntu 22.04: usa `qtbase5-dev` (paso 1). |
| El `.deb` falla con dependencias no satisfechas | Descargaste el `.deb` de otra versión de Ubuntu, o estás en 24.04. Usa el de tu versión o la opción A. |
| `cmake` no encuentra ARGoS al compilar tus propios controladores | ARGoS no está instalado en el sistema (`sudo make install`), o quedaron dos instalaciones mezcladas: desinstala ambas y deja una sola. |
