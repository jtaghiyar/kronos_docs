======
Guides
======

.. _quick_tutorial:

Quick tutorial
===============
This section can help you quickly make a component, make a pipeline, and run the pipeline step by step using somatic variant caller `Strelka <https://sites.google.com/site/strelkasomaticvariantcaller/>`_. You need to refer to the rest of the documentation for details.

First, you need to download the ``strelka_workflow.tar.gz`` tarball from our ftp server: ``ftp://ftp.bcgsc.ca/public/shahlab/kronos``, and unpack it into a desired directory, say ``$HOME_DIR``:

.. code:: bash

    $ cd $HOME_DIR
    $ tar -xvf strelka_workflow.tar.gz

This will make a directory called ``strelka_workflow`` in ``$HOME_DIR``.

The test input data is a lightweight pair of cell line tumour/normal bam files that can be downloaded from here:

 * `Exome normal bam file <https://xfer.genome.wustl.edu/gxfer1/project/gms/testdata/bams/hcc1395/gerald_C1TD1ACXX_7_CGATGT.bam>`_
 * `Exome tumour bam file <https://xfer.genome.wustl.edu/gxfer1/project/gms/testdata/bams/hcc1395/gerald_C1TD1ACXX_7_ATCACG.bam>`_
    
We are using this lightweight pair of bam files so that this can be done on a local computer with decent amount of memory ( >=8G).


Make a component
^^^^^^^^^^^^^^^^
We are going to make a component called *plot_strelka* which will later be used in the next section.
The seed of this component is an R script called ``plot_strelka.R`` which is included in the tarball in the ``seeds`` directory.
It takes a Strelka result file as an input and generates a plot.

.. note::

    Refer to the :ref:`Components <components>` for more information on the definition of *seed* and *component*.

The seed can be run by the following command:  
 
 .. code:: bash
 
     $ R --no-save --args <infile> <outfile_name> < plot_strelka.R

where ``<infile>`` and ``<outfile_name>`` should be replaced with a Strelka result file and a name for the output file, respectively. 

We can make a component for this seed by following steps 1 to 6: 

**Step 1.** Make a component template:

 .. code:: bash

     $ kronos make_component plot_strelka
    
 This will make a component template called *plot_strelka* in the current working directory. 


**Step 2.** Copy the seed to the ``plot_strelka/component_seed`` directory.

      
**Step 3.** Open the ``plot_strelka/component_main.py`` template file and add the following lines to the beginning of the ``make_cmd`` function:

 .. code:: python
 
        cmd = self.requirements['R'] + ' --no-save --args '
        cmd_args = [self.args.infile, self.args.outfile_name]
        cmd_args.append('<')
        cmd_args.append(os.path.join(self.seed_dir, 'plot_strelka.R'))  

**The above four lines are the only codes that we need to write for this component.**

Essentially, the above four lines create the same command that is used to run the seed, *i.e.* ``$ R --no-save --args <infile> <outfile_name> < plot_strelka.R``.   

Generally, in the ``component_main.py`` of a component, we always only need to create the command that is used to run its seed.

For this component, we have chosen '``R --no-save --args``' as the command ``cmd`` in the first line, and the other three lines simply list the the rest of the command.
Also, the following lines in the template file created by ``Kronos`` are not required for this component and therefore can be deleted or commented out:
 
 .. code:: python
  
        args = vars(self.args)
        comp_seed_map = {
                         #e.g. 'component_param1': 'seedParam1',
                         #e.g. 'component_param2': 'seedParam2',
                        }

        for k, v in args.items():
            if v is None or v is False:
                continue

            ## TODO: uncomment the next line if you are using
            ## comp_seed_map dictionary.
            # k = comp_seed_map[k]            
            
            cmd_args.append('--' + k)
            
            if isinstance(v, bool):
                continue
            if isinstance(v, str):
                v = repr(v)
            if isinstance(v, (list, tuple)):
                cmd_args.extend(v)
            else:
                cmd_args.extend([v])

Therefore the final ``component_main.py`` would look like:
 
 .. code:: python
 
     """ 
     component_main.py
     This module contains Component class which extends 
     the ComponentAbstract class. It is the core of a component.

     Note the places you need to change to make it work for you. 
     They are marked with keyword 'TODO'.
     """

     from kronos.utils import ComponentAbstract
     import os


     class Component(ComponentAbstract):
    
         """
         TODO: add component doc here. 
         """

         def __init__(self, component_name="plot_strelka", 
                      component_parent_dir=None, seed_dir=None):
        
             ## TODO: pass the version of the component here.
             self.version = "v0.99.0"

             ## initialize ComponentAbstract
             super(Component, self).__init__(component_name, 
                                        component_parent_dir, seed_dir)

         ## TODO: write the focus method if the component is parallelizable.
         ## Note that it should return cmd, cmd_args.
         def focus(self, cmd, cmd_args, chunk):
             pass 
     #        return cmd, cmd_args

         ## TODO: this method should make the command and command arguments 
         ## used to run the component_seed via the command line. Note that 
         ## it should return cmd, cmd_args. 
         def make_cmd(self, chunk=None):
             ## TODO: replace 'comp_req' with the actual component
             ## requirement, e.g. 'python', 'java', etc.
             cmd = self.requirements['R'] + ' --no-save --args '
             cmd_args = [self.args.infile, self.args.outfile_name]
             cmd_args.append('<')
             cmd_args.append(os.path.join(self.seed_dir, 'plot_strelka.R')) 
        
             if chunk is not None:
                 cmd, cmd_args = self.focus(cmd, cmd_args, chunk)
            
             return cmd, cmd_args

     ## To run as stand alone
     def _main():
         c = Component()
         c.args = component_ui.args
         c.run()

     if __name__ == '__main__':
         import component_ui
         _main()
     
.. The rest of the steps do not need any programming and simply prepare the input and requirements.
  
**Step 4.** As mentioned earlier, the seed takes as an input a Strelka result file and a name for the output file.
We have chosen the names ``infile`` and ``outfile_name`` to represent these inputs, respectively.
So, open the ``plot_strelka/component_params.py`` template file and add the names to it as follows:

 .. code:: bash
 
     """
     component_params.py

     Note the places you need to change to make it work for you. 
     They are marked with keyword 'TODO'.
     """

     ## TODO: here goes the list of the input files. Use flags: 
     ## '__REQUIRED__' to make it required
     ## '__FLAG__' to make it a flag or switch.
     input_files  = {
                     'infile' : '__REQUIRED__', 
     #                 'input_file2' : None
                     }

     ## TODO: here goes the list of the output files.
     output_files = {
                     'outfile_name' : '__REQUIRED__',
     #                 'output_file1' : None
                     }

     ## TODO: here goes the list of the input parameters excluding input/output files.
     input_params = {
     #                 'input_param1' : '__REQUIRED__',
     #                 'input_param2' : '__FLAG__',
     #                 'input_param3' : None
                     }

     ## TODO: here goes the return value of the component_seed. 
     ## DO NOT USE, Not implemented yet!
     return_value = []

.. note::

    You only need to change the following two lines:
    
    .. code:: python
    
        'infile' : '__REQUIRED__',
        'outfile_name' : '__REQUIRED__',
         
**Step 5.** Open the ``plot_strelka/component_reqs.py`` template file and only change the following line:

 .. code:: python
 
     requirements = {
     #                'python': '__REQUIRED__',
                    }
 
to this:
 
 .. code:: python

     requirements = {
                      'R': '__REQUIRED__',
                    } 
                    
The rest of the fields in this file can also be changed if desired but is not required.    

**Step 6 (Optional).** This step can be skipped. It is only needed if you want to run the component as standalone outside of a pipeline.
This step creates a user interface for the component.
Open the ``plot_strelka/component_ui.py`` template file and change it so it looks like:

 .. code:: bash
 
     """
     component_ui.py

     Note the places you need to change to make it work for you. 
     They are marked with keyword 'TODO'.
     """

     import argparse

     #==============================================================================
     # make a UI 
     #==============================================================================
     ## TODO: pass the name of the component to the 'prog' parameter and a
     ## brief description of your component to the 'description' parameter.
     parser = argparse.ArgumentParser(prog='plot_strelka', 
                                      description = """
                                      creates a plot from Strelka results.""")

     ## TODO: create the list of input options here. Add as many as desired.
     parser.add_argument(
                         "--infile", 
                         default = None,
                         required = True, 
                         help= """
                         input file.
                         """)
                    
     parser.add_argument(
                         "--outfile_name", 
                         default = None,
                         required = True, 
                         help= """
                         a name for the output file.
                         """)

     ## parse the argument parser.
     args, unknown = parser.parse_known_args()
     
Make a pipeline
^^^^^^^^^^^^^^^
This section explains how to create a pipeline that runs `Strelka <https://sites.google.com/site/strelkasomaticvariantcaller/>`_ and plots its results.
For this purpose, we need to use the components included in the ``strelka_workflow.tar.gz`` tarball. 
There are two components called ``run_strelka`` and ``plot_strelka`` in ``$HOME_DIR/strelka_workflow/components`` where ``$HOME_DIR`` is where you unpacked the tarball.
You can also use the component we created in `Make a component`_ for ``plot_strelka``.

Next, export the components path to ``PYTHONPATH`` environment variable:

.. code:: bash

    export PYTHONPATH=$HOME_DIR/strelka_workflow/components:$PYTHONPATH

Now, we can start making a new pipeline using these component:
 
**Step 1.** Make a new configuration file using the ``make_config`` command:

.. code:: python

    kronos make_config run_strelka plot_strelka -o strelka_workflow
    
This will create a new configuration file called ``strelka_workflow.yaml`` in the current working directory.
This file has a number of sections which we go through step by step.
Please refer to :ref:`Configuration file <config_file>` to learn more about each section.


**Step 2 (optional).** The first section in the configuration file is ``__PIPELINE_INFO__`` and contains information regarding the pipeline. 
Here is an example for this configuration file:

 .. code:: bash
 
     name: 'run_plot_strelka'
     version: '1.0'
     author: 'Jafar Taghiyar'
     data_type: 'SNV'
     input_type: 'bam'
     output_type: 'vcf, jpeg'
     host_cluster: 'local'
     date_created: '2016-01-04'
     date_last_updated: 
     Kronos_version: '2.0.4'

This section is only informative and does not have any effects on the pipeline.

**Step 3.** The second section is ``__GENERAL__`` that lists the requirements of all the components in the pipeline. The requirements listed in the ``__GENERAL__``  section apply to all the components in the pipeline. 

In this pipeline, it looks like this:

.. code:: bash
 
     strelka: '__REQUIRED__'
     R: '__REQUIRED__'
     perl: '__REQUIRED__'

These entries are required. 
However, these values can come from a setup file when running the pipeline (see `How to run the pipeline`_).
So, for now you do not need to pass values to them in the configuration file.

.. topic:: Info

    Each component can also have its own requirements specified individually. However, in this quick tutorial you need to simply leave them blank. For more information refer to :ref:`here <_task_requirements>`.

**Step 4.** The next section is ``__SHARED__`` where we can create variables. 

In this pipeline, we add the following variable to this section:

.. code:: bash

     __SHARED__:
         strelka_ref: #a reference genome

Similar to ``__GENERAL__`` section, the value for this entry can come from the setup file when running the pipeline (see `How to run the pipeline`_).
In **Step 6**, we will see how we use it.


**Step 5.** Next is ``__SAMPLES__`` section that can be used to list the input files or parameters.
By default the section looks like:

.. code:: bash

    __SAMPLES__:
        # sample_id:
        #    param1: value1
        #    param2: value2
    
In this pipeline, the input files are a pair of tumour/normal bam files.
Also, we choose a parameter from Strelka component called ``min_tier2_mapq`` to included as input in this section to show the functionality of the section.

The content of this section can be provided in an input file when running the pipeline (see `How to run the pipeline`_).
So, a user does not need to pass values here.

In **Step 6**, we will see how we use the content of this section.

**Step 6.** The rest of the configuration file contains ``__TASK__`` sections.
These sections are where the connections among different components in the pipeline are specified.
We also need to pass proper values to all the parameters in these sections that have ``__REQUIRED__`` keyword as input.
For example, in this pipeline, we have the following entries with ``__REQUIRED__`` as their values that we need to pass actual values to:

* in ``__TASK_1__`` section:

 .. code:: bash
 
     tumour: __REQUIRED__
     ref: __REQUIRED__
     normal: __REQUIRED__
     output_dir: __REQUIRED__
     
* in ``__TASK_2__`` section:

 .. code:: bash
 
     infile: __REQUIRED__ 
     outfile_name: __REQUIRED__
      
Some of these entries will be filled when specifying the flow of the pipeline.
For example, we like to get the input for the first task from an input file, *i.e.* from ``__SAMPLES__`` section.
For this purpose, we use :ref:`IO connections <connections>`:

.. code:: bash

    __TASK_1__:
    .
    .
        component:
            input_files:
                tumour: ('__SAMPLES__', 'tumour')
                .
                .
                normal: ('__SAMPLES__', 'normal') 

Next, we want the second task, ``__TASK_2__``, to get its input from the output of the first task, ``__TASK_1__``.
Therefore, we simply pass the name of the output file from Strelka in the first task to the ``infile`` parameter of the second task:

.. code:: bash

    __TASK_2__:
    .
    .
        component:
            input_files:
                infile: passed.somatic.snvs.vcf
 
and then we add the ``__TASK_1__`` to the ``forced_dependecies`` of ``__TASK_2__`` which makes ``__TASK_2__`` to wait for ``__TASK_1__`` to finish first:

.. code:: bash

    __TASK_2__:
    .
    .
        run:
        .
        .
            forced_dependencies: ['__TASK_1__']
            
.. note::

    Since Strelka software enforces the name of its result file to be *passed.somatic.snvs.vcf*, we need to use the exact same name in the configuration file and then use the ``forced_dependencies``.
    Otherwise, an :ref:`IO connection <connections>` could help automatically pass the output of one task to the input of another task.
    It also manages the dependencies automatically.

So far, We have already passed desired values to ``tumour``, ``normal``, and ``infile``.
For ``output_dir`` and ``outfile_name`` we only need to pick names.
Let's choose ``strelka_output`` and ``results/passed.somatic.snvs.pdf`` for them, respectively:

.. code:: bash

    __TASK_1__:
    .
    .
        component:
        .
        .
            output_files:
                output_dir: strelka_output
    __TASK_2__:
    .
    .
        component:
        .
        .
            output_files:
                outfile_name: results/passed.somatic.snvs.pdf
            
.. note::

    The ``results/`` in ``results/passed.somatic.snvs.pdf`` instructs ``Kronos`` to make a directory called ``results`` and copy the result file ``passed.somatic.snvs.pdf`` there.
    
Since we want to enable a user to pass a reference genome in the setup file, *i.e.* without having to change the configuration file, we pass it as a variable in ``__SHARED__`` section (see **Step 4**) and use a connection to refer to it in the configuration file: 

.. code:: bash

    __TASK_1__:
    .
    .
        component:
            input_files:
            .
            .
                ref: ('__SHARED__','strelka_ref')

All the connections will be automatically replaced in the runtime.

**Step 7.** In **Step 5**, we chose the ``min_tier2_mapq`` parameter to be included in the ``__SAMPLES__`` section.
Therefore, in order to use it we need to add a connection as follows:

.. code:: bash

    __TASK_1__:
    .
    .
        component:
            parameters:
            .
            .
                min_tier2_mapq: ('__SAMPLES__','mapq2')

.. note:: 

    ``mapq2`` in the above line, is only an arbitrary key that we choose and it can be a different name.
    This key is used in the input file when running the pipeline later in `Run a pipeline`_.
    
The final configuraion file looks like this:

.. code:: bash

    __PIPELINE_INFO__:
        name: 'run_plot_strelka'
        version: '1.0'
        author: 'Jafar Taghiyar'
        data_type: 'SNV'
        input_type: 'bam'
        output_type: 'vcf, jpeg'
        host_cluster: 'local'
        date_created: '2016-01-04'
        date_last_updated: 
        Kronos_version: '2.0.4'
    __GENERAL__:
        strelka: '__REQUIRED__'
        R: '__REQUIRED__'
        perl: '__REQUIRED__'
    __SHARED__:
        strelka_ref: #a reference genome
    __SAMPLES__:
        # sample_id:
        #    param1: value1
        #    param2: value2

    __TASK_1__:
        reserved:
            # do not change this section.
            component_name: 'run_strelka'
            component_version: '1.2.0'
            seed_version: '1.0.13'
        run:
            # NOTE: component cannot run in parallel mode.
            use_cluster: True
            memory: '10G'
            num_cpus: 1
            forced_dependencies: []
            add_breakpoint: False
            env_vars:
            boilerplate:
            requirements:
                strelka:
                perl:
        component:
            input_files:
                tumor: "('__SAMPLES__', 'tumour')"
                config:
                ref: "('__SHARED__','strelka_ref')"
                normal: "('__SAMPLES__', 'normal')"
            parameters:
                skip_depth_filters: "('__SHARED__','strelka_exome')"
                min_tier1_mapq: 20
                num_procs: 8
                min_tier2_mapq: "('__SAMPLES__','mapq2')"
            output_files:
                output_dir: 'strelka_output'
    __TASK_2__:
        reserved:
            # do not change this section.
            component_name: 'plot_strelka'
            component_version: '0.99.0'
            seed_version: '0.99.0'
        run:
            # NOTE: component cannot run in parallel mode.
            use_cluster: True
            memory: '5G'
            num_cpus: 1
            forced_dependencies: ['__TASK_1__']
            add_breakpoint: False
            env_vars:
            boilerplate:
            requirements:
                R:
        component:
            input_files:
                infile: 'passed.somatic.snvs.vcf'
            parameters:
            output_files:
                outfile_name: 'results/passed.somatic.snvs.pdf'
    
Run a pipeline 
^^^^^^^^^^^^^^
In this section, we are going to run the simple tumour/normal pair single nucleotide variant calling pipelnie that we made in `Make a pipeline`_.
This pipeline consists of two tasks:

* *task 1:* runs `Strelka
  <https://sites.google.com/site/strelkasomaticvariantcaller/>`_ on a
  pair of tumour and normal bam files.
* *task 2:* creates a series of plots from Strelka output.

Requirements
************
* Python >= v2.7.6
* Strelka == v1.0.14
* Java >= v1.7.0_06
* Perl >= v5.8.8+

How to run the pipeline
***********************
.. comment:
 **Step 1.** Export the path of the components to ``PYTHONPATH``, i.e.:

 .. code:: bash
  
      export PYTHONPATH=$HOME_DIR/strelka_workflow/components:$PYTHONPATH
      
**Step 1.** Create a file called ``setup.txt`` with the following content:

.. code:: bash
 
    #section key value
    __GENERAL__ strelka <path to /strelka_install_dir/bin/configureStrelkaWorkflow.pl>
    __GENERAL__ R <path to R executable, e.g. R> 
    __GENERAL__ perl <path to perl executable, e.g. perl>
    __SHARED__ strelka_ref $HOME_DIR/strelka_workflow/refs/GRCh37.75.fa

.. note::

    The above file is a tab separated file and the first line, *i.e.* '``#section key value``', is part of the file.
    Also the *value* column should be replaced with actual values.
    Please note that a reference genome is included in the ``strelka_workflow.tar.gz`` tarball that can be used for ``strelka_ref``.
    The ``$HOME_DIR`` should be replaced with the directory where the tarball is unpacked.
    
**Step 2.** Create a file called ``input.txt`` with the following content:

.. code:: bash
 
    #sample_id tumour normal mapq2
    SAM_0 <tumour bam file path> <normal bam file path> 0
    SAM_5 <tumour bam file path> <normal bam file path> 5

.. note::

    The above file is a tab separated file and the first line, *i.e.* '``#sample_id tumour normal mapq2``', is part of the file.
    ``SAM_0`` and ``SAM_5`` are arbitrary ID's.
    However, the ID's cannot be used more than once in an input file, *i.e.* they must be unique.
    
    For tumour and normal bam files, you can use the lightweight pair mentioned in the begining of this document.
    If you already have a pair of bam files, then you can use them too. 
    However, the plots will be different than what we present here. 
    
**Step 3.** Run the pipeline using the following command:

.. code:: bash
 
     kronos run -c $HOME_DIR/strelka_workflow/components/ -e strelka 
                -i input.txt -r TEST_RUN -s setup.txt -w RES 
                -y <path to strelka_workflow.yaml> --no_prefix

Please note to replace ``<path to strelka_workflow.yaml>`` with the actual path.

Outputs
*******
The resulting files will be saved in the current direcory under ``RES`` subdirectory.
For this pipeline, the final output files are:

* RES/TEST_RUN/SAM_0/outputs/results/passed.somatic.snvs.pdf
* RES/TEST_RUN/SAM_1/outputs/results/passed.somatic.snvs.pdf

Please refer to :ref:`results directory structure <results_dir>` for more information on the outputs directory.

If you have used the lightweith pair of cell line data, then the plots should look like the following (NOTE: we have put all the resulting plots in one single plot for the sake of illustration):



 .. figure:: strelka_portrait_cellLine.png
     :width: 750px
     :align: center
     :height: 650px
     :alt: alternate text

More Examples
^^^^^^^^^^^^^
Please refer to our Github `repositories <https://github.com/MO-BCCRC?tab=repositories>`_ for more examples. The repositories with the postfix ``_workflow`` are the pipelines and the rest are the components.

.. _deploy_kronos_to_the_cloud:

Deploy Kronos to the cloud
==========================

This section will present the steps needed to get started with running your very own cloud cluster.

Setup StarCluster
^^^^^^^^^^^^^^^^^

In this guide, we will be using Amazon Web Services (AWS) and `StarCluster <http://star.mit.edu/cluster/index.html>`_ for creating cloud clusters. 
StarCluster manages the instantiation of EC2 instances and the installation of the compute grid scheduler. 
Here, we will be using Sun Grid Engine (SGE). StarCluster also provides many other useful features such as elastic load balancing,  which dynamically resizes your cloud cluster depending on the load. 
This allows you to optimize either cost (by reducing the number of nodes as soon as they are no longer needed) or time (by adding more nodes as needed).

Installing StarCluster
**********************

Much of this section is taken from the `Quick Start <http://star.mit.edu/cluster/docs/latest/quickstart.html>`__ guide from StarCluster's documentation. 
If anything isn't clear, I recommend you look there for more information.

We will start by installing StarCluster from PyPI.

.. code:: bash

    $ pip install starcluster

Once installed, we need to do some preliminary setup. 
First, we need to create a configuration file for StarCluster. 
After running ``starcluster help``, select option 2.

.. code:: bash

    $ starcluster help
    StarCluster - (http://star.mit.edu/cluster) (v. 0.95.6)
    Software Tools for Academics and Researchers (STAR)
    Please submit bug reports to starcluster@mit.edu

    !!! ERROR - config file /home/bgrande/.starcluster/config does not exist

    Options:
    --------
    [1] Show the StarCluster config template
    [2] Write config template to /home/bgrande/.starcluster/config
    [q] Quit

    Please enter your selection:

Second, you need to add in your AWS credentials. 
You can follow the instructions `here <http://docs.aws.amazon.com/general/latest/gr/getting-aws-sec-creds.html>`__ to find them in the AWS Console. 
Once you have both your access and secret keys as well as your account user ID, you can paste them under ``[aws info]`` in your StarCluster configuration.

.. code:: bash

    $ vi ~/.starcluster/config
    # Add in AWS credentials under [aws info]
    [aws info]
    aws_access_key_id = #your aws access key id here
    aws_secret_access_key = #your secret aws access key here
    aws_user_id = #your 12-digit aws user id here

Third, you need to create a key pair, which will allow you to remotely login via SSH to your cloud clusters without a password.

.. code:: bash

    $ starcluster createkey starcluster_key -o ~/.ssh/starcluster_key.rsa

Fourth, you can now configure StarCluster to use that new key pair by default.

.. code:: bash

    $ vi ~/.starcluster/config
    # Add a new key pointing to the one you just created
    [key starcluster_key]
    key_location = ~/.ssh/starcluster_key.rsa
    # Then, configure the smallcluster template to use that key
    [cluster smallcluster]
    keyname = starcluster_key

Lastly, we need to change the default AMI used for creating EC2 instances. 
The default AMIs that StarCluster offers are a bit dated, with the latest one running Ubuntu 13. 
We have created an updated AMI running Ubuntu 14.04 LTS with Kronos pre-installed.

The latest AMI for running Kronos is: ``ami-0f326465``.

Additionally, this image requires instances that support hardware virtual machine (HVM) images. 
This allows for special features such as `Enhanced Networking <https://aws.amazon.com/ec2/instance-types/#enhanced_networking>`__.
Therefore, we are also going to update the instance type.

.. code:: bash

    $ vi ~/.starcluster/config
    # Change the default AMI to the one above and the 
    # instance type to m3.medium (for testing purposes)
    [cluster smallcluster]
    NODE_IMAGE_ID = ami-97efa3fd
    NODE_INSTANCE_TYPE = m3.medium

Creating an EBS volume
**********************

It's useful to have a volume that is automatically mounted to the cloud cluster when launched that persists after cluster termination.
Otherwise, you need to make sure you download the data before terminating your instance. 
It also allows you to have large volumes, which is necessary when dealing with sequencing datasets such as in cancer genomics.

.. warning:: 

    Because EBS volumes persist after cluster termination, they will continue to cost you. 
    Be sure not to forget about them. 

StarCluster offers handy commands for creating new EBS volumes. 
Here, we are creating a 1-TB volume called "awesome\_study\_volume". This process can take a while, depending on the size of your volume; it took 17 minutes when I ran it. 
Notice that we're shutting down the instance after volume creation, as we won't need it again for now.

.. code:: bash

    $ starcluster createvolume --name= awesome_study_volume --shutdown-volume-host 1000 us-east-1c

.. note::
    
    Unfortunately, StarCluster doesn't support the creation of the newer SSD EBS volumes (gp2), which supports higher performance and sizes greater than 1 TB. 
    If you need either of these, you can `create a volume <http://docs.aws.amazon.com/AWSEC2/latest/UserGuide/ebs-creating-volume.html>`__ using the Console interface.

Next, you're gonna want to configure StarCluster to mount this volume on your cluster. 
Make sure to note the new volume ID after running the previous step (e.g., vol-278402da).

.. code:: bash

    $ vi ~/.starcluster/config
    # Add the newly created volume to your configuration
    [volume awesome_study_volume]
    VOLUME_ID = vol-278402da
    MOUNT_PATH = /projects/

We don't want to necessarily mount this volume on every cloud cluster we instantiate. 
Therefore, we will create a separate cluster template based on the ``smallcluster`` template as follows.

.. code:: bash

    $ vi ~/.starcluster/config
    # Add new cluster template that extends smallcluster
    [cluster awesome_study_config]
    EXTENDS = smallcluster
    VOLUMES = awesome_study_volume

Launching your cloud cluster
****************************

We're finally ready to launch your cloud cluster! For this, you have one simple command to run. 
It will take a few minutes for everything to setup.

.. code:: bash

    # Create a new cloud cluster named awesome_study_cluster
    # based on the awesome_study_config template.
    $ starcluster start --cluster-template awesome_study_config awesome_study_cluster

Setup Kronos
^^^^^^^^^^^^

After you are done setting up your cloud cluster, you can remotely login using your key pair without having to enter a password.

.. code:: bash

    # SSH into your cloud cluster's master node
    $ starcluster sshmaster awesome_study_cluster

The root Python (``/usr/bin/python``) already has Kronos' dependencies installed in addition to Kronos itself.

.. code:: bash

    $ kronos --help
    usage: kronos [-h] [-w WORKING_DIR] [-v]
                  {make_component,make_config,update_config,init,run} ...

    Kronos: a workflow assembler for cancer genome analytics and informatics

    positional arguments:
      {make_component,make_config,update_config,init,run}
        make_component      make a template component
        make_config         make a config file
        update_config       copy the fields of old config file to new config file.
        init                initialize a pipeline from the given config file
        run                 run kronos-made pipelines w/o initialization.

    optional arguments:
      -h, --help            show this help message and exit
      -w WORKING_DIR, --working_dir WORKING_DIR
                            path of the working dir
      -v, --version         show program's version number and exit

Running Kronos from this point on is standard. 
The only details worth noting is the following argument values when launching a Kronos pipeline.

::

    # This is where the drmaa library is located
    --drmaa_library_path /opt/sge6/lib/linux-x64/libdrmaa.so
    # Since this cloud cluster uses SGE, you can use the following option
    --job_scheduler sge
    # The parallel environment on this cluster is called orte
    --qsub_options '-pe orte {num_cpus} -l mem_free={mem} -l h_vmem={mem}'
