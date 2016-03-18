========================================================================
kronos: A workflow assembler for cancer genome analytics and informatics
========================================================================
``kronos`` is a software tool for automating reproducible, auditable and distributable bioinformatics workflow development.
Kronos obviates explicit coding for workflow development to a great extent by compiling a text configuration file into an executable Python scripts.
Each resulting workflow includes a run manager to automatically execute the workflow locally, on a cluster or in the cloud, to parallelize tasks, and to log all runtime events.
The workflows are highly modular and configurable by construction, facilitating flexible and extensible workflows which can be modified easily through configuration file editing.

``kronos`` is a free and open source Python package under the MIT license and is shipped with both a Docker image and an Amazon machine image.

.. This document has the following major parts: 

.. - *Part I:* covers details about the features and how to use module 
.. - *Part II:* describes the config file 
.. - *Part III:* explains how to make new components 
.. - *PartIV:* explains how to launch a pipeline

 
Table of Contents
=================

.. toctree::
   :maxdepth: 5
   
   Getting started <getting_started/getting_started>
   Package <kronos_package/package>
   Configuration file <config_file/config_file>
   Components <components/components>
   Pipelines <launch_pipeline/run>
   Guides <guides/guides>
   License <license/LICENSE>
   Contacts <contact/contacts.rst>
..   Versions <version/version>

.. Indices and tables
.. ==================

.. * :ref:`genindex`
.. * :ref:`modindex`
.. * :ref:`search`

.. _YAML: http://yaml.org/


.. topic: Citation

   please cite us by:
   <ref to the paper goes here>

.. topic:: Author

    M. Jafar Taghiyar jafar.taghiyar@gmail.com

.. topic:: Date last updated

    2016-03-18
