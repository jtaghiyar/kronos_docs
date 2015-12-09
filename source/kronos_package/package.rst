========================
kronos package
========================

This section explains:

- :ref:`kronos features <pf_features>`
- :ref:`kronos commands <kronos_commands>`

.. _pf_features:

``kronos`` features
=============================
``kronos`` package offers the following features which eliminate the difficulties of making a pipeline:

.. topic:: Info

    We define a *pipeline* as a `DAG <http://en.wikipedia.org/wiki/Directed_acyclic_graph>`_ composed of different :ref:`tasks <task_sec>`.

- :ref:`Single configuration file <config_file>`: the whole pipeline can be configured using a single configuration file
- :ref:`Parallelization <parallelization>`: parallelizable tasks are automatically run in parallel
- :ref:`Synchronization <synchronization>`: parallel tasks can be synchronized based on any of their parameters
- :ref:`Local, cluster and cloud support <cloud>`: the pipelines can be run locally or on a cluster of computing nodes or on cloud
- :ref:`Forced dependencies <forced_dependencies>`: any task can be forced to wait for any other tasks
- :ref:`Breakpoints <breakpoint>`: a pipeline can be programmatically paused and restated from any point in the pipeline
- :ref:`Boilerplates <boilerplate>`: an executable boilerplate or script can be injected to a task and is run prior to running the task itself
- :ref:`Keywords <config_file_keywords>`: a set of specific keywords in the configuration file which will be automatically replaced by proper values in the run time
- :ref:`Parameter sweep <samples_sec>`: a pipeline can be run for a list of different values for a set of input arguments
- :ref:`Output directory customization <output_dir_customization>`: the structure of the output directory where all the results are stored can be configured by the user
- :ref:`Event logging <>`: all the events are automatically logged


.. _kronos_commands:

``kronos`` commands
=====================
Once kronos is installed, it is added to the PATH, i.e. ``kronos`` becomes an available command which has the following sub-commands:

.. csv-table:: 
    :header: "Command", "Description"
    :widths: 15, 65

    "``make_component``", "make a new component template"    
    "``make_config``", "make a new configuration file "
    "``update_config``", "copy the fields of old configuration file to new configuration file"
    "``init``", "initialize a pipeline from the given configuration file "
    "``run``", "run kronos-made pipelines with optional initialization"


as well as the following options:

.. csv-table:: 
    :header: "Options", "Description"
    :widths: 20, 40
    
    "**-h** or **--help**", "print help - optional"
	"**-v** or **--version**", "show program's version number and exit - optional"
	"**-w** or **--working_dir**", "path/to/working_dir - optional"

.. topic:: Tip 
    
    The ``-w`` is optional and if not specified, the current working directory is used to save output files/directories.
    It is recommended to specify it to avoid overwriting existing files.
    See :ref:`What is the working directory? <working_dir>` for more information.
    
.. _make_component:

``make_component``
**************************
This command creates a new component template. 
In other words, it automatically generates wrappers required for a *seed* to become a *component*.

.. topic:: Info

    See :ref:`components` for more information on seed and component.

The command is used as follows:

.. code-block:: bash
    
    kronos -w </path/to/working_dir> make_component <name_for_component>

For example, the following code creates a component template called ``my_comp`` in a directory called ``my_components_dir``:

.. code-block:: bash
    
    kronos -w my_components_dir make_component my_comp

.. _make_config:

``make_config`` 
***********************
This command makes a new :ref:`configuration file <config_file>` for the given list of component names.

The command is used as follows:

.. code-block:: bash

    kronos -w </path/to/working_dir> make_config <list_of_components> -o <name_for_config_file>

For example, the following code creates a new configuration file called :file:`my_config_file.yaml` for two components ``comp1`` and ``comp2`` in a directory called ``my_working_dir``:

.. code-block:: bash

    kronos -w my_working_dir make_config comp1 comp2 -o my_config_file

.. warning::

    It is required to export the path of the :ref:`components directory <components_dir>` to the ``PYTHONPATH`` environment variable prior to running the ``make_config`` command:

    .. code-block:: bash
    
        export PYTHONPATH=</path/to/components_dir>:$PYTHONPATH
        
.. topic:: Tip 

    Note that the suffix ``.yaml`` is automatically added to the end of the provided name for the configuration file.
   
``update_config``
***********************
This command replaces the corresponding fields of an old configuration file with that of a new one.
This is useful when there is a large configuration file which needs to be updated.

The command is used as follows:

.. code-block:: bash

    kronos -w </path/to/working_dir> update_config <old_config.yaml> <new_config.yaml> -o <output_filename>

For example, the following code creates a new configuration file called :file:`new_config_file.yaml` by updating ``my_config_file1.yaml`` using ``my_config_file2.yaml`` in a directory called ``my_working_dir``:

.. code-block:: bash

    kronos -w my_working_dir update_config my_config_file1.yaml my_config_file2.yaml -o new_config_file

.. _init:

``init`` 
*************************
This command initializes a new pipeline (i.e. creates a Python script) based on the input configuration file.

.. topic:: Info

    We call a resulting Python script a *pipeline script* too.
    
The command is used as follows:

.. code-block:: bash

    kronos -w </path/to/working_dir> init -y </path/to/config_file.yaml> -e <name_for_pipeline>

For example, the following code creates a Python script called :file:`my_pipeline.py` for the input configuration file :file:`my_config_file.yaml` in a directory called ``my_working_dir``: 

.. code-block:: bash

    kronos -w my_working_dir init -y my_config_file.yaml -e my_pipeline

The output Python script of this command can be run using kronos :ref:`run <run>` command or can be run directly as a Python script.
   
.. topic:: Info

   See :ref:`How to initialize a pipeline? <how_to_init_pipeline>` for more information.

.. topic:: Tip

    Note that the suffix ``.py`` is automatically added to the end of the provided name for the pipeline.

.. warning:: 

    The ``init`` command might create the following directories in addition to the pipeline Python script:

    - intermediate_config_files
    - intermediate_pipeline_scripts

    These directories are used by ``kronos`` and users should NOT modify them.

.. _run:

``run`` 
****************
This command runs kronos-made pipelines, i.e. pipeline scripts made by ``init`` command.

The command is used as follows:

.. code-block:: bash
    
    kronos run -k </path/to/my_pipeline_script.py> -c </path/to/components_dir> [options]

.. topic:: Info

    You can use ``run`` command to initialize and run the pipeline using the configuration file directly (i.e. without the need to ``init`` first).
    See :ref:`Run the pipeline using run command <how_to_run_pipeline>` for more information. 
    

