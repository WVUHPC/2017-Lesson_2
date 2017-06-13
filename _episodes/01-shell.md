---
title: "Bash Scripting"
teaching: 45
exercises: 15
questions:
- "Key question"
objectives:
- "First objective."
keypoints:
- "First key point."
---

## Shell scripting to manipulate files and folders

Lets explore the use of shell scripts for a common case for researchers.

LAMMPS is a classical molecular dynamics code that models an ensemble of
particles in a liquid, solid, or gaseous state.
In particular we want to explore a cracking in 2D and the influence of
the mass of the particles on the region of fracture.

You do not need to understand the details of creating input files for LAMMPS.
The input file looks like this (the file is called "in.crack")

~~~
# 2d LJ crack simulation

dimension	2
boundary	s s p

atom_style	atomic
neighbor	0.3 bin
neigh_modify	delay 5

# create geometry

lattice		hex 0.93
region		box block 0 100 0 40 -0.25 0.25
create_box	5 box
create_atoms	1 box

mass		1 1.0
mass		2 1.0
mass		3 1.0
mass		4 1.0
mass		5 1.0

# LJ potentials

pair_style	lj/cut 2.5
pair_coeff	* * 1.0 1.0 2.5

# define groups

region	        1 block INF INF INF 1.25 INF INF
group		lower region 1
region		2 block INF INF 38.75 INF INF INF
group		upper region 2
group		boundary union lower upper
group		mobile subtract all boundary

region		leftupper block INF 20 20 INF INF INF
region		leftlower block INF 20 INF 20 INF INF
group		leftupper region leftupper
group		leftlower region leftlower

set		group leftupper type 2
set		group leftlower type 3
set		group lower type 4
set		group upper type 5

# initial velocities

compute	  	new mobile temp
velocity	mobile create 0.01 887723 temp new
velocity	upper set 0.0 0.3 0.0
velocity	mobile ramp vy 0.0 0.3 y 1.25 38.75 sum yes

# fixes

fix		1 all nve
fix		2 boundary setforce NULL 0.0 0.0

# run

timestep	0.003
thermo		200
thermo_modify	temp new

neigh_modify	exclude type 2 3

dump		1 all atom 500 dump.crack

dump		2 all image 400 image.*.jpg type type &
		zoom 1.6 adiam 1.5
dump_modify	2 pad 4

#dump		3 all movie 250 movie.mpg type type &
#		zoom 1.6 adiam 1.5
#dump_modify	3 pad 4

run		9000
~~~
{: .source}

Lets first run the simulation with the input as it is now.
As usual we create a submission script. LAMMPS have executables for running in
serial and parallel. The job is too small to justify using the parallel version.
We will just request 1 core on 1 node. Everything else is very standard.

~~~
#!/bin/sh

#PBS -N LAMMPS
#PBS -l nodes=1:ppn=1
#PBS -l walltime=00:05:00
#PBS -m ae
#PBS -q debug

module purge
module load compilers/gcc/6.3.0 mpi/openmpi/2.0.2_gcc63 atomistic/lammps/2016.11.17

cd $PBS_O_WORKDIR

lmp_serial_png < in.crack
~~~
{: .source}

Submit the job with

~~~
qsub runjob.pbs
~~~
{: .source}

The job takes less than a minute to complete. When finished you will see a few
image files.
If you have enable X Window on your local machine you can see the image with

~~~
display image.8000.jpg
~~~
{: .source}

Now, this is the challenge:
Imagine that you want to study the influence of the mass of the particles on
the cracking region, the original mass for those atoms is declared with the input line

~~~
mass		1 1.0
~~~
{: .source}

Now, imagine that you want to run jobs for the value of the mass changing to
2.5, 5.0, 7.5 and 10.0
We would like to have a way of creating an script that will create directories
for jobs and changing the input accordingly.
This exercise allow us to present several concepts useful for such kind of
research.

~~~
#!/bin/bash
for i in 2.5 5.0 7.5 10.0
do
    dirname=MASS_$i
    if [ -d $dirname  ]
    then
        echo "Directory $dirname exists!"
    else
        mkdir $dirname
    fi
    if [ -a $dirname/runjob.pbs ]
    then
        echo "File $dirname/runjob.pbs exists"
    else
        ln -s ../runjob.pbs -t $dirname
    fi
    cat in.crack | sed "s/\(mass\s*1\s*\) 1.0/\1 $i/g" > $dirname/in.crack

done
~~~
{: .source}

This script shows a number of elements in shell scripts.
How to create variables, the use of for loops and conditionals.
How to test for the existence of directories and files.
Finally, the script shows how to use sed and regular expressions to modify files.

### Exploring two parameters

This script explores a more complex case where two variables will be changed.

~~~
#!/bin/bash

i=1
while [ $i -lt 21 ]
do
    mass=`echo "1.0 + ($i * 0.2)" | bc`
    j=1
    while [ $j -lt 21 ]
    do
        sigma=`echo "1.0 + ($j * 0.01)" | bc`

        dirname=MASS_${mass}_SIGMA_${sigma}
        if [ -d $dirname  ]
        then
            echo "Directory $dirname exists!"
        else
            mkdir $dirname
        fi
        if [ -a $dirname/runjob.pbs ]
        then
            echo "File $dirname/runjob.pbs exists"
        else
            ln -s ../runjob.pbs -t $dirname
        fi
        cat in.crack | sed "s/\(mass\s*1\s*\) 1.0/\1 $mass/g" \
        | sed "s/pair_coeff\s*\* \* 1.0 1.0 2.5/pair_coeff \* \* $sigma 2.5/g" \
        > $dirname/in.crack

        j=$((j+1))
    done
    i=$((i+1))
done
~~~
{: .source}


{% include links.md %}
