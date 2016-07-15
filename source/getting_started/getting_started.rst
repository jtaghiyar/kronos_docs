===============
Getting started
===============

Download
========
You can get the ``Kronos`` package from `PyPi <https://pypi.python.org/pypi/kronos-pipeliner>`_.
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

where ``drmaa-python`` is optional and you will need to install it only if you want to use ``-b drmaa`` when running a workflow on a cluster.

.. topic:: Info

    ``PyYaml`` and ``ruffus`` are installed as dependencies when installing ``Kronos``. 
    So, you would only need to install ``drmaa-python``.

Install
=======
You can install ``Kronos`` using ``pip``:

.. code-block:: bash

    pip install kronos-pipeliner

or upgrade it by:

.. code-block:: bash

    pip install --upgrade kronos-pipeliner


.. topic:: Tip

    For a quick start, without having to go through all the details, you can directly refer to :ref:`Quick tutorial <quick_tutorial>`.

.. toctree::
..   :maxdepth: 2


