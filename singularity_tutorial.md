## Exercise: Running Containers with Singularity on RMACC Summit

### Logging into RMACC Summit

#### _Option 1:_ If you are using a temporary account: In a terminal or Git Bash window, type the following:
```
ssh -X userNNNN@tlogin1.rc.colorado.edu
Password: <temporary-password-you-received-from-me>
```
_Note that `userNNNN` is a temporary account that I have assigned to you (where "NNNN" is a number like "0012"). If you do not have a temporary account or password, let me know._

#### _Option 2:_ If you are using your regular RC account (e.g., ‘monaghaa’ for me), log in as you normally would, e.g.:
 ```
ssh -X monaghaa@login.rc.colorado.edu
```

### Getting on a compute node

Navigate to scompile
```
ssh scompile
```

Now start an interactive job:
```
sinteractive -n 2 -t 90 --reservation=container
```

Load singularity and set up environment
```
module load singularity/3.3.0
export SINGULARITY_TMPDIR=/scratch/summit/$USER
export SINGULARITY_CACHEDIR=/scratch/summit/$USER 
```

### Running a container from Singularity Hub

 Pull an existing container image that someone else posted:
```
singularity pull --name mytranslator.sif shub://monaghaa/mytranslator
```

…And run it:
```
singularity run mytranslator.sif
```

 …And look at the script inside:
```
singularity inspect --runscript mytranslator.sif
```

### Running a container from Docker Hub

 Now let’s grab the stock docker python container:
```
singularity pull --name pythond.sif docker://python
```

 …And run python from it:
```
singularity exec pythond.sif python
```

…And shell into the container and look around:
```
singularity shell pythond.sif
```

 …try `ls /` What directories do you see?:

Let’s run an external python script using the containerized version of python: 
First create a script called “myscript.py” as follows:
```
echo 'print("hello world from the outside")' >myscript.py
```

…And now let’s run the script using the containerized python
```
singularity exec pythond.sif python ./myscript.py
```

…Conclusion: Scripts and data can be kept inside or outside the container. In some instances (e.g., large datasets or scripts that will change frequently) it is easier to containerize the software and keep everything else outside.

### Binding directories to a container

On Summit, most host directories are “bound” (mounted) by default. But on other systems, or in some instances on Summit, you may want to access a directory that is not already mounted.
Let’s try it:

Note that the “/opt” directory in ”pythond.sif” is empty. But the Summit ”/opt” directory is not.  Let’s bind it:
```
singularity shell --bind /opt:/opt pythond.sif
```

Now from within the container type "ls -l /opt" and see if it matches what you see from the outside of the container if you type the same thing.

 …It isn’t necessary to bind like-named directories like we did above. Try binding your /home/$USER directory to /opt.
```
singularity shell --bind /home/$USER:/opt pythond.sif
```

Now from within the container type "ls -l $HOME" and see if it matches
what you see from the outside of the container if you type the same thing.

_Note: If your host system does not allow binding, you will need to create the host directories you want mounted when you build the container (as root on, e.g., your laptop)

### Running an MPI container

MPI-enabled Singularity containers can be deployed on RMACC Summit, with the caveat that the MPI software within the container must be consistent with MPI software available on the system. This requirement diminishes the portability of MPI-enabled containers, as they may not run on other systems without compatible MPI software. Regardless, MPI-enabled containers can still be a very useful option in many cases.   

Here we provide an example of that uses a gcc compiler with OpenMPI.  Let’s pull it from Dockerhub first:

```
singularity pull hello_openmpi.sif shub://monaghaa/hello_openmpi_summit
```

In order to use it, we load the gcc and openmpi modules on Summit (these are consistent with the gcc/openmpi versions installed in the container)
```	
module load gcc/6.1.0
module load openmpi/2.0.1
```

To run it, simply preface the ‘singularity exec <stuff>’ command with ‘mpirun –n <numprocs>’:

```
mpirun -n 2 singularity exec hello_openmpi.sif mpi_hello_world
