---
layout: post
title:  "Julia: Install and first steps"
categories: Julia
tags:  julia
author: Nico
---
* content
{:toc}

Guía para instalar Julia en Linux (Ubuntu).

Se una la terminal para instalar dentro de un direcctorio deseado.
Además se da tips de como usar Julia con Atom. Y como agregar el Kernel de Julia en Jupyter Notebook.




## Instalar Julia

Julia is dynamically typed, feels like a scripting language, and has good support for interactive use.

### Installation

Descargar

- **Método 1:** descargar directamente de: https://julialang.org/downloads/ . Escoger de acuerdo al sistema operativo y la arquitectura que se tenga.

- **Método 2:** Usar `wget`

  ```
  cd ~/Downloads # Navegar hasta Downloads
  ```

  ```
  wget https://julialang-s3.julialang.org/bin/linux/x64/1.5/julia-1.5.2-linux-x86_64.tar.gz
  ```

Descomprimir:

```
tar -xvzf julia-1.5.2-linux-x86_64.tar.gz
```

Copiar a la dirección deseada, en mi caso:

```
cp -r julia-1.5.2 /home/nico/
```

Agregar Julia al Path

```
sudo nano -c /etc/environment
```

Agregar los siguiente: `/home/nico/julia-1.5.2/bin`

Exportar el Path

```
export PATH=$PATH:/home/nico/julia-1.5.2/bin
source /etc/environment
```

Ejecutar Julia

```
julia
```

### Julia con Atom

Abrir Atom Setings>Packages y tipear uber-juno e instalar. Este paquete instalará:

- julia-user
-

### Julia en Jupyter

Requisitos

- Tener instalado python o Anaconda (conda)
- Tener instalado jupyter

Desde terminal de Julia

```julia
using Pkg
Pkg.add("IJulia")
```
