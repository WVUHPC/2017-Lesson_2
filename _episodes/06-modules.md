---
title: "Creating environmental modules"
teaching: 45
exercises: 15
questions:
- "Key question"
objectives:
- "First objective."
keypoints:
- "First key point."
---

## Introduction

Environment Modules (EM) provides a way for the dynamic modification of a user's environment via modulefiles.
The Environment Modules package is a tool that simplify shell initialization and lets users easily modify their environment during the session.
To achieve its goal, EM uses files called modules located on special locations, you can load and unload modules, changing the environment variables.

## Creating a module for fftw double precision

For this tutorial we will create a private repository for your own modules and we will create one module for the library we just create in our previous section.

The firs step is decide a place where we we locate the modules. For this tutorial we will use `$HOME/local/modules`. The location is arbitrary as soon as you can write in that directory. The first step is to create that directory.

The next step is to setup the variable `MODULEPATH` on your `.bashrc` pointing to the place where you will add your modules. Open you favorite text editor and edit your \verb|$HOME/.bashrc|. You can add this at the very end of the file. Assuming you use bash:

~~~
export MODULEPATH=$HOME/local/modules:$MODULEPATH
~~~
{: .source}

In order to make effective the changes you need to "source" the file,
incorporating the changes in your current session.

~~~
source $HOME/.bashrc
~~~
{: .source}

Now it is time to create your first module.
Create a file into your `$HOME/local/modules` folder.
The name of the file should be something that indicates what is special
with the module. Lets call it for example

~~~
fftw3-long-double_gcc63
~~~
{: .source}

Using that name you know that the module is for double precision and was
compiled with GCC 6.3

~~~
#%Module1.0##########################################
## Fast Fourier Transform Library
##

module-whatis	"Name: fftw"
module-whatis	"Version: 3.3.6"
module-whatis	"Category: C subroutine library"
module-whatis	"Description: Library for computing the discrete Fourier transform (DFT)"
module-whatis	"URL: http://fftw.org/"

set  prefix     <ENTER_YOUR_HOME_DIR_HERE>/local

# This is used during compilation for searching for libraries
prepend-path  LIBRARY_PATH        ${prefix}/lib

# This is used during execution for searching for libraries
prepend-path  LD_LIBRARY_PATH     ${prefix}/lib

# This is used during compilation for searching headers (*.h and *.mod)
prepend-path  CPATH               ${prefix}/include

# These two are usual places where man pages and info pages are located
prepend-path  INFOPATH            ${prefix}/share/info:
prepend-path  MANPATH             ${prefix}/share/man

# This is a search path for searching for executables
prepend-path  PATH                ${prefix}/bin

# This is a helper tool used when compiling applications and libraries.
# It helps you insert the correct compiler options on the command line
prepend-path  PKG_CONFIG_PATH     ${prefix}/lib/pkgconfig
~~~
{: .source}

See if the module appears on your list of available modules

~~~
module avail
~~~
{: .source}

Load the module with

~~~
module load fftw3-long-double_gcc63
~~~
{: .source}

Loading the module will change variables that allow you to compile the code from
the previous session like this:

~~~
gcc example_fftl.c -lfftw3l -lm
~~~
{: .source}

Notice that both `-I$HOME/local/include` and `-L$HOME/local/lib` are not longer
needed, variables `LIBRARY_PATH` and `CPATH` contains the paths to libraries and
headers respectively such that gcc knows where to search for those things.

{% include links.md %}
