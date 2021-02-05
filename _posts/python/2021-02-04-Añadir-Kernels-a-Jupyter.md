---
layout: post
title:  "Añadir Kernels a Jupyter en Windows"
categories: Python
tags:  jupyter kernel
author: Nicol Huaraca
---
* content
{:toc}

Guía para instalar diferentes kernels en Jupyter Notebook o Jupyter Lab.

Permite usar Julia, R, Octave y Matlab en formato markdown mediante el editor Jupyter.



## R

- Instalar R

Descargar R del siguiente link [The Comprehensive R Archive Network (r-project.org)](https://cloud.r-project.org/) 

- Instalar los paquetes necesarios

```R
install.packages(‘devtools’)
devtools::install_github(‘IRkernel/IRkernel’)
```

- Activar kernel para jupyter desde R (ojo que estamos usando python no conda)

```
IRkernel::installspec()
```

Existe una forma de permitir que el kernel de R esté disponible para todos los usuarios: `IRkernel::installspec(user = FALSE)`, sin embargo en esta modalidad necesitas tener permiso de super usuario o administrador (abrir el Command Promt como administrador).

**Nota:** Si desea instalar una versión de kernel diferente al 4.0.3, usar el siguiente comando:

```R
# Para R 3.6
IRkernel::installspec(name = ‘ir33’, displayname = ‘R 3.6’)
# Para R 3.2
IRkernel::installspec(name = ‘ir32’, displayname = ‘R 3.2’)
```

## Julia

- Instalar Julia

  Descargar Julia de [Download Julia (julialang.org)](https://julialang.org/downloads/) para 64 bits o 32 bits dependiendo de su sistema. Para comprobar su instalación abrir Julia

<img src="/assets/images/julia.PNG" style="zoom:50%;" />

- Instalar kernel desde Julia

```julia
using Pkg
Pkg.add("IJulia")
```

## Octave

- Instalar Octave

  Descargar el archivo `.exe` de Octave del siguiente link: [Download (gnu.org)](https://www.gnu.org/software/octave/download.html#ms-windows) para 64 0 32 bits. e instalar.  

  **Nota:** Si lo que se quiere es Octave para poder usar `Dynare`, la versión que se debe descargar es Octave 5.2.0

- Agregar el Kernel de Octave

  Primero se debe agregar `octave-cli` al path. Por lo que se debe agregar la siguiente dirección (`"C:\Octave\Octave-5.2.0.0\mingw64\bin"`) a las variables de entorno virtual. Para ello buscar `environment variables` en el buscador y abrir:

<img src="/assets/images/env1.png" align= "center" style="zoom:50%;" />

<img src="/assets/images/env2.PNG" align= "center" style="zoom:40%;" />

Seleccionar Editar y luego Nuevo y agregar la ruta mencionada. Por último desde instalamos el  kernel de Octave desde python:

```python
pip install octave_kernel
```



## Manejando kernels

- Obtener lista de kernels

```python
jupyter kernelspec list
```

- Remover algún kernel

```python
jupyter kernelspec uninstall julia-3.2
```



