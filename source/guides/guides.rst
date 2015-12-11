======
Guides
======

Deploying Kronos to the Cloud (Amazon AWS)
==========================================

Setup StarCluster
-----------------

In this guide, we will be using Amazon Web Services (AWS) and StarCluster for creating cloud clusters. 
StarCluster manages the instantiation of EC2 instances and the installation of the compute grid scheduler. 
Here, we will be using Sun Grid Engine (SGE). StarCluster also provides many other useful features such as elastic load balancing,  which dynamically resizes your cloud cluster depending on the load. 
This allows you to optimize either cost (by reducing the number of nodes as soon as they are no longer needed) or time (by adding more nodes as needed).

Installing StarCluster
~~~~~~~~~~~~~~~~~~~~~~

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

The latest AMI for running Kronos is: ``ami-97efa3fd``.

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

Creating an EBS Volume
~~~~~~~~~~~~~~~~~~~~~~

It's useful to have a volume that is automatically mounted to the cloud cluster when launched that persists after cluster termination.
Otherwise, you need to make sure you download the data before terminating your instance. 
It also allows you to have large volumes, which is necessary when dealing with sequencing datasets such as in cancer genomics.

.. warning:: 

    As I said, EBS volumes persist after cluster termination.
    Therefore, they continue to cost money as long as they exist. 
    Don't forget about them.

StarCluster offers handy commands for creating new EBS volumes. 
Here, we are creating a 1-TB volume called "awesome\_study\_volume". This process can take a while, depending on the size of your volume; it took 17 minutes when I ran it. 
Notice that we're shutting down the instance after volume creation, as we won't need it again for now.

.. code:: bash

    $ starcluster createvolume --name= awesome_study_volume --shutdown-volume-host 1000 us-east-1c

.. warning::
    
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

Launching Your Cloud Cluster
~~~~~~~~~~~~~~~~~~~~~~~~~~~~

We're finally ready to launch your cloud cluster! For this, you have one simple command to run. 
It will take a few minutes for everything to setup.

.. code:: bash

    # Create a new cloud cluster named awesome_study_cluster
    # based on the awesome_study_config template.
    $ starcluster start --cluster-template awesome_study_config awesome_study_cluster

Setup Kronos
------------

Once you remotely login into your cloud cluster, you'll notice that Kronos is already installed for you. 
The root Python (``/usr/bin/python``) already has Kronos' dependencies installed.

.. code:: bash

    # SSH into your cloud cluster's master node
    $ starcluster sshmaster awesome_study_cluster
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
