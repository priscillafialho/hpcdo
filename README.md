hpcdo
=====

command line tool to ease the interaction with remote cluster and their schedulers.

<a href="https://www.gittip.com/tlamadon/">
  <img alt="Support via Gittip" src="https://rawgithub.com/twolfson/gittip-badge/0.2.0/dist/gittip.png"/>
</a>

## features

 - get a snapshot of available resources
 - get a list of currently running tasks
 - easily get a view of the output of currently running task (including error log)
 - easily schedule new tasks using configuration files
 - easily sync local/remote folder, update git repository on remote servers

## install

Install using pip or easy_install

    pip install hpcdo

## under the hood

The library first has a description file for the jobs which allows the user to control then using names.
The configuration file is writen in YAML and should be present in working folder. Secondly the class uses a file
to keep information about running tasks. When a task is launched, its id is stored with the associated information.
This allows to tail, delete or clear the task by name instead of using the job id.

The library when submitting a new job does:

 1. loads the job description from the YAML file (name, log files, template file)
 2. apply the value from YAML to the template
 3. creates a temporary qsub file
 4. submits using qsub and capture job id
 5. appends the job to the job list for later use

## Notes

This python packages helps dealing with submitting jobs to the hpc and tracking them. It facilitates the wrting of
qsub script with tend to be quite obscure and repetitive. It also helps keeping track of all the files generated by a
given script. You will be able to tail, clear, or backup the output fo given run of a given job.

So a job has a description that includes a name, the type of the job (stata or julia) and also information
about where the logs should go, and the number of nodes on which the job should run. Because a given job can
be ran several time, the user will have to provide a name when running more than 1 at a time.

A command line interface is provided, but the library can also be used from inside shovel.py for example.

## Quickstart in dev mode

### install

you need to use python with some local packages, so first add the following to your environment (`~/.bash_profile`):

    export PYTHONPATH='<abs_path_to_home>/usr/local/lib/python2.6/site-packages/'

add the path so that you can find binary automatically also in `~/.bash_profile`:

    export PATH=$PATH:/data/uctptla/usr/local/bin

install a few python packages, at the command line:

    easy_install --prefix=<abs_path_to_home>/usr/local multitail jinja2 pyaml

get this repository and install the `hpcdo` package in develop mode, at the command line:

    python setup.py develop --prefix=/data/uctptla/usr/local

### setting it up

now from this point, you should be able to run the following:

    hpcdo -h

then in your directory where you run your qsub files, you can create some qsub templates, for instance something like
the following `cmd.qsub` file:

    #!/bin/bash

    echo "starting qsub script file for fibonacci timer"
    source ~/.bash_profile
    date

    module load sge/2011.11

    # here's the SGE directives
    # ------------------------------------------
    #$ -q batch.q      # <- the name of the Q you want to submit to
    #$ -pe mpich {{ncore}}  #  <- load the openmpi parallel env w/ $(arg1) slots
    #$ -S /bin/bash    # <- run the job under bash
    #$ -N {{name}}     # <- name of the job in the qstat output
    #$ -o {{logout}}   # direct output stream to here
    #$ -e {{logerr}} # <- name of the stderr file.
    #$ -wd {{wd}}

    echo "calling mpirun now"

where the objects in between curly brackets will be replace at runtime. You complement this with a job
description file `hpc_jobs.yaml`, for example:

    job1:
      name:   example
      desc:   simple parallel test
      ncore: 5
      logout:   example.log
      logerr:   example.err
      template: cmd.qsub
      wd: /data/uctptla/git/Examples/julia/sge


### using it

now you can do the following:

    hpcdo ls              # list available jobs
    hpcdo sub job1        # submits job1
    hpcdo sub job1 -n 20  # submits job1 with a different number of cores
