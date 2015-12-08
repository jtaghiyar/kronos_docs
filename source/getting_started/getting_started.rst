===============
Getting started
===============

Download
========
You can get the ``kronos`` package from `PyPi <https://pypi.python.org/pypi>`_.
You can also clone it from `GitHub repository <https://github.com/jtaghiyar/kronos>`_.

.. The *components* can also be cloned from the github repository `here <https://svn.bcgsc.ca/stash/projects/PF/repos/pipeline-components/browse>`_.

Dependencies
============
You need to have the following dependencies installed:

.. csv-table:: 
    :header: "Program/package", "Version"
    :widths: 20, 10
    
    "python", "2.7.*"
    "ruffus", "2.4.1"
    "PyYaml", "3.11"
    "drmaa-python", "0.7.6"

You can simply create a file called ``requirements.txt`` that looks like this:

.. code-block:: rest

    pyyaml>=3.11
    ruffus==2.4.1
    drmaa>=0.7.6

which can be used when installing the package.

Install
=======
You can install ``kronos`` using ``pip``:

.. code-block:: bash

    pip install kronos_pipeliner -r requirements.txt

.. toctree::
..   :maxdepth: 2


