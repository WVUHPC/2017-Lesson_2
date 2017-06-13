---
title: "Building and installing software"
teaching: 45
exercises: 15
questions:
- "How to build my own software packages?"
objectives:
- "First objective."
keypoints:
- "First key point."
---

## Introduction

Sometimes, the modules available on the system are not enough or you need to compile the code by yourself to get some extra functionality not present on the current modules.

For this tutorial we will go on the whole process of compiling a code by yourself.
We have selected fftw a well known library for computing Fast Fourier Transforms.

First, we go to the webpage of the FFTW code \url{http://www.fftw.org}

Before downloading the code, create a directory on your home folder suitable for compiling codes, for example `"$HOME/local/src"` and go into such directory.

Go to downloads and copy the link to the code that we will download and copy the link.

On the terminal execute:

~~~
wget http://www.fftw.org/fftw-3.3.6-pl2.tar.gz
~~~
{: .source}


Now that you have downloaded the code uncompress it using the command line

~~~
tar -zxvf fftw-3.3.6-pl2.tar.gz
~~~
{: .source}


Now go into that directory. There is usually a file `README` or `INSTALL`. Those files will give you instructions on how to compile the code.
We need to load the module for GCC 6.3 using the command line

~~~
module load compilers/gcc/6.3.0
~~~
{: .source}

It is a good idea to create a directory for building the code.
Here we will use `build_gcc`.
Go into that directory and execute:

~~~

../configure --help
~~~
{: .source}


You will see all the options available for configure the code. System administrators are usually conservative when choosing options for compilation, usually shifting the preference towards stability rather than performance.
Consider for example the case where we want the long-double precision library rather than the original double precision.
The configuration line will be like this:

~~~
../configure --prefix=$HOME/local --enable-long-double
~~~
{: .source}


The next step is compile the code with

~~~
make
~~~
{: .source}


It is always good practice to test the compilation. Good software comes with tests that compare results with known results.

~~~
make check
~~~
{: .source}


Finally, the installation is done with:

~~~
make install
~~~
{: .source}


Now, we have fftw for long-double precision compiled and installed at \verb|$HOME/local|. Check by yourself the existence of folders such as lib and include, they contain both the libraries and headers needed to compile other programs using the library you just compiled.

We have a small program to test the FFT library we just compiled. The code is as follows:

~~~
#include <stdio.h>
#include <math.h>
#include <fftw3.h>

#define NUM_POINTS 10000
#define REAL 0
#define IMAG 1

void create_input(fftwl_complex* signal) {
  /* The input is a sum of several cosines and sines with different frequencies
   * and amplitudes
   */
  int i;

  printf("Creating a signal with precision LONG DOUBLE (sizeof=%lu Bytes)\n", sizeof(long double));

  for (i = 0; i < NUM_POINTS; ++i) {
    long double theta = (long double)i / (long double)NUM_POINTS * 2 * M_PI;

    signal[i][REAL] = 1.0 * cos(10.0 * theta) +
      2.0 * cos(20.0 * theta) +
      3.0 * cos(30.0 * theta) +
      4.0 * cos(40.0 * theta) +
      5.0 * cos(50.0 * theta);

    signal[i][IMAG] = 1.0 * sin(10.0 * theta) +
      2.0 * sin(20.0 * theta) +
      3.0 * sin(30.0 * theta) +
      4.0 * sin(40.0 * theta) +
      5.0 * sin(50.0 * theta);
  }
}

void print_magnitude(fftwl_complex* result, FILE *fp) {
  int i;
  for (i = 0; i < NUM_POINTS; ++i) {
    long double mag = sqrt(result[i][REAL] * result[i][REAL] +
			   result[i][IMAG] * result[i][IMAG]);
    fprintf(fp,"%34.25Le %34.25Le %34.25Le\n", result[i][REAL], result[i][IMAG], mag);
  }
}

int main() {
  FILE *fp;
  fftwl_complex signal[NUM_POINTS];
  fftwl_complex result[NUM_POINTS];

  fftwl_plan plan = fftwl_plan_dft_1d(NUM_POINTS,
				      signal,
				      result,
				      FFTW_FORWARD,
				      FFTW_ESTIMATE);

  create_input(signal);
  printf("Saving input signal...\n");
  fp = fopen("Input_FFT.dat", "w");
  print_magnitude(signal, fp);
  fclose(fp);

  fftwl_execute(plan);

  printf("Saving FFT from signal...\n");
  fp = fopen("Output_FFT.dat", "w");
  print_magnitude(result, fp);
  fclose(fp);

  fftwl_destroy_plan(plan);
  return 0;
}
~~~
{: .source}

To compile this code you have to be very explicit on the locations of the libraries and headers because they are no included in the environmental variables that gcc uses to search for them. The compilation line will be:

~~~
gcc example_fftl.c -I$HOME/local/include -L$HOME/local/lib -lfftw3l -lm
~~~
{: .source}


When output is not declared like above, the executable is a file called `a.out`.
We have the advantage that we produce a static library for the long-double precision version of FFTW. The library is `libfftw3l.a`. The big advantage of static libraries is that the application can be certain that all its libraries are present and that they are the correct version. Being static for FFTW, our example simply runs with:

~~~
./a.out
~~~
{: .source}


You can check library dependencies executing

~~~
ldd ./a.out
~~~
{: .source}


Once executed you will have 2 files: `Input_FFT.dat` 	and	`Output_FFT.dat`, those files contain the original signal and its Fourier-Transform function. We create a small python script to help you visualize both functions. The functions are in complex space, so you are drawing Real and Imaginary parts and the magnitude of the signal.

In the next section we will see how we can create modules that will facilitate compilation and execution of binaries that need these libraries.

## Exercises:

* Compile the FFTW in their single precision (float) and double precision versions. You can use the same place `$HOME/local` as prefix for your installing the libraries. It is always good idea to clean the build directory before trying to compile a new version.

* Compile FFTW enabling the build of shared libraries.
Try to compile the same code using them. Check dependencies with `ldd` and notice how the execution is not longer possible without explicitly adding the environmental variable `LD_LIBRARY_PATH` pointing to the location of the libraries.


{% include links.md %}
