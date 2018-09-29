
# Jupyterlab tutorial

#### The advantages of running Jupyterlab on [Kamiak](https://hpc.wsu.edu/users-guide/):
+ Labs can utilize scripts which access data files from a centralized location. This makes using version control (like [github](https://github.com/)) much easier because every version of the script will utilize the same path to input files and output files

+ Jupyterlab can be used for parallel computing.

+ It is convenient to analyze very large datafiles when they are housed on the cluster instead of on a local machine.

+ Jupyterlab provides a graphical interface to Kamiak reducing the amount of work necessary to get started using the cluster.

+ The use of notebooks increases reproducibility by encouraging clear and consistent code commenting as well as ggregating complicated workflows into a single location.

+ Jupyterlab can utilize the [R](https://www.datacamp.com/community/blog/jupyter-notebook-r) kernel!

## To get started
+ Request access to [Kamiak](https://hpc.wsu.edu/staff-contact-info/) 
+ Connect to Kamiak using an ssh terminal. Tutorial [here](https://hpc.wsu.edu/users-guide/terminal-ssh/).

### To be completed once only:
1. Create a new script called ***jupyterlab.sh*** on Kamiak


```bash
cd ~
nano jupyterlab.sh
```

+ Paste the following code block into this new file
+ Save the file by entering ***ctrl-x***, choosing ***Yes*** when prompted and finally hitting ***enter*** to confirm the name of the file.


```bash
#!/bin/bash
#SBATCH --partition=cas
#SBATCH --nodes=1
#SBATCH --ntasks-per-node=1
#SBATCH --time 12:00:00
#SBATCH --job-name jupyterlab
#SBATCH --output jupyterlab.out
#SBATCH --error jupyterlab.err

# get tunneling info
node=$(hostname -s)
user=$(whoami)
address=$user"@wsu.edu"
subject="Jupyterlab tunneling instructions"
cluster="kamiak.wsu.edu"

# load modules or conda environments here
module load anaconda3
module load r

# Run Jupyter
XDG_RUNTIME_DIR=""
jupyter-lab --no-browser --ip=${node} --notebook-dir=/ &> jupyterlab.log &

#  Wait until the server started successfuly and identify assigned port and security token
link=""
while [ -z "${link}" ]; do
    sleep 1
    link=$(grep -m 1 'http://' jupyterlab.log | grep -Eo 'http://.*' )
done

port=$(echo "$link" | grep -oP '(?<=:).*?(?=/)') 
    
# print tunneling instructions to jupyterlab.out
echo -e "Instructions to connect to Jupyterlab
script provided by: The Busch Lab

Hello $user,

STEP 1: Open a ssh tunnel to Kamiak. 
A ssh tunnel provides a secure communication path between local and remote computers.
Kamiak has already opened the tunnel on the server end. To open the tunnel on the local side 
enter one of the following commands in either a terminal window (mac,linux) or command 
prompt (windows). Keep this window open. Closing it will also close the local end of 
the tunnel. Don't forget to close the remote end by running scancel on the jupyterlab 
job when you are done.
 
For mac/linux:
ssh -N -L ${port}:${node}:${port} ${user}@${cluster}

For windows (make sure PuTTY is installed):
plink.exe -N -L ${port}:${node}:${port} ${user}@${cluster}

STEP 2: Connect to the JupyterLab server.
Enter the following address in a web browser such as firefox or chrome. 

${link//$node/localhost}

Have fun with the science!
"

# Email contents jupyterlab.out to the current user
mailx -s "$subject" -S smtp=smtp://smtp.wsu.edu $address < jupyterlab.out

wait
```

### To be completed every time: 
To start the jupyterlab server running, log on to Kamiak and enter the following from your home directory.


```bash
sbatch jupyterlab.sh
```

This will start the jupyterlab server running on a compute node. The script will then email you instructions on how to connect to the server once it is up and running.

# Optional
Jupyterlab notebooks provide a clean way to separate data analysis workflow into blocks of markdown followed by blocks of code. Code blocks are then executed utilizing an engine known as a **kernel**. By default jupyterlab comes with the python3 kernel. However it is possible to add a large number of other kernels permitting notebooks to be written in a wide variety of languages (R, Fortran, Bash, Julia, Spark, ruby, C++, Go, nodeJS, PHP, MATLAB, Wolfram Mathematica, Java, ect...) For a list of available kernels see the [wiki](https://github.com/jupyter/jupyter/wiki/Jupyter-kernels). Instructions on installing the R and BASH kernels are included below:

## To make the R kernel available to jupyterlab notebooks:
Once connected, open a terminal window from within jupyterlab (file->new->terminal) and enter the following commands **one line at a time**.


```bash
R
install.packages('devtools', dependencies=TRUE)
devtools::install_github('IRkernel/IRkernel')
IRkernel::installspec()
```

Now the R kernel will be available ***the next time you start the jupyterlab server***

## To make the BASH kernel available to jupyterlab notebooks:
Once connected, open a terminal window from within jupyterlab (file->new->terminal) and enter the following commands one line at a time. Note the ***--user*** option on the first line. This flag circumvents the need for root access when installing python packages on kamiak. Note: you don't need this kernel to use BASH Python has packages that allow you to interact with BASH and you can access a terminal window from within Jupyter. You only really need this option if all of the code in your notebook will be written in BASH.


```bash
pip install bash_kernel --user
python -m bash_kernel.install
```

Now the BASH kernel will be available ***the next time you start the jupyterlab server***
