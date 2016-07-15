========================================================================
Kronos: A workflow assembler for cancer genome analytics and informatics
========================================================================
``Kronos`` is a highly flexible Python-based software tool that mainly enables bioinformatics developers, *i.e.* bioinformaticians who develop workflows for analyzing genomic data, to quickly make a workflow.
It uses Ruffus_ as the underlying workflow management system and adds a level of abstraction on top of it, which significantly reduces programming overhead for workflow development and provides a mechanism to represent a workflow by a top level YAML_ configuration file.

Each resulting workflow is portable, can run either locally or on a cluster, parallelizes tasks automatically, and logs all runtime events.
The workflows are also highly modular and can be easily updated by editing their corresponding configuration files.

``Kronos`` is free and open source under the MIT license and has a `Docker image <https://hub.docker.com/r/jtaghiyar/kronos/>`_ too. Although we have developed it with bioinformaticians in mind, Kronos can be used in any area where a workflow is required to run a series of tasks on a given set of input data.

.. note::

    Throughout this documentation we will use *workflow* and *pipeline* interchangeably.

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

.. _Ruffus: http://www.ruffus.org.uk/
.. _YAML: http://yaml.org/


.. topic:: Citation:

   Please cite Kronos using `this paper <http://dx.doi.org/10.1101/040352>`_.

.. topic:: Date last updated this document:

    2016-07-13
