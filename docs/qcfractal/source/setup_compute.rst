Setup Compute
=============

Once a QCFractal server is running, compute can be attached to it by spinning
up ``qcfractal-manager``. These ``qcfractal-manager`` connect to your
FractalServer instance, adds tasks to a distributed workflow manager, and
pushes complete tasks back to the ``qcfractal-server`` instance. These
``qcfractal-manager`` should be run on either the machine that is executing
the computations or on the head nodes of supercomputers and local clusters.


Distributed Workflow Engines
----------------------------

QCFractal supports a number of distributed workflow engines to execute
computations. Each of these has strengths and weaknesses depending on the
workload, task specifications, and resources that the compute will be executed
on. In general, we recommend the following:

- For laptops and single nodes: ProcessPoolExecutor
- For local clusters: Dask

The ProcessPoolExecutor uses built-in Python types and requires no additional
libraries while Dask requires ``dask``, ``dask.distributed``, and
``dask_jobqueue``.

Using the Command Line
----------------------

At the moment only ProcessPoolExecutor ``qcfractal-manager`` can be spun up
from the command line as other distributed workflow managers require
additional setup.

Launching a ``qcfractal-manager`` using a ProcessPoolExecutor:

.. code-block:: console

    $ fractal-manager executor
    [I 190301 10:45:50 managers:118] QueueManager:
    [I 190301 10:45:50 managers:119]     Version:         v0.5.0

    [I 190301 10:45:50 managers:122]     Name Information:
    [I 190301 10:45:50 managers:123]         Cluster:     unknown
    [I 190301 10:45:50 managers:124]         Hostname:    qcfractal.local
    [I 190301 10:45:50 managers:125]         UUID:        0d2b7704-6ac0-4ef7-b831-00aa6afa8c1c

    [I 190301 10:45:50 managers:127]     Queue Adapter:
    [I 190301 10:45:50 managers:128]         <ExecutorAdapter client=<ProcessPoolExecutor max_workers=8>>

    [I 190301 10:45:50 managers:131]     QCEngine:
    [I 190301 10:45:50 managers:132]         Version:    v0.6.1

    [I 190301 10:45:50 managers:150]     Connected:
    [I 190301 10:45:50 managers:151]         Version:     v0.5.0
    [I 190301 10:45:50 managers:152]         Address:     https://localhost:7777/
    [I 190301 10:45:50 managers:153]         Name:        QCFractal Server
    [I 190301 10:45:50 managers:154]         Queue tag:   None
    [I 190301 10:45:50 managers:155]         Username:    None

    [I 190301 10:45:50 managers:194] QueueManager successfully started. Starting IOLoop.

The connected ``qcfractal-server`` instance can be controlled by:

.. code-block:: console

    $ qcfractal-manager --fractal-uri=api.qcfractal.molssi.org:80

.. note::

    The ``--name`` argument is useful for change the name of the manager
    reported back to the ``qcfractal-server`` instance. In addition, the
    ``--queue-tag`` will limit the acquisition of tasks to only the desired
    ``qcfractal-server`` task tags.

Using the Template Generator
----------------------------

Due to the complexity of setting up various distributed workflow managers a
``qcfractal-template`` CLI is available to generate ``qcfractal-manager``
scripts that will both setup a distributed workflow system and a ``qcfractal-
manager``.

To setup a Dask with the SLURM queue manager the following command can be run:

.. code-block:: console

    $ qcfractal-template parsl slurm
    $ cat manager_template.py

    """
    Dask Distributed Manager Helper

    Conditions:
    - Dask Distributed and Dask Job Queue (dask_jobqueue in Conda/pip)
    - Manager running on the head node
    - SLURM manager

    For additional information about the Dask Job Queue, please visit this site:
    https://jobqueue.dask.org/en/latest/
    """

    # Fractal Settings
    # Location of the Fractal Server you are connecting to
    FRACTAL_URI = "localhost:7777"
    ...

The generated script has a small tutorial on how to correct fill in the
relevant data.


.. note::

    This is a temporary solution to this complex problem, we will be moving to
    configuration files in the future.


Using the Python API
--------------------


``qcfractal-managers`` can also be created using the Python API.

.. note::

    This is for advanced users and special care needs to be taken to ensure
    that both the manager and the workflow tool need to understand the number
    of cores and memory available to prevent oversubscription of compute.

.. code-block:: python

    from qcfractal.interface import FractalClient
    from qcfractal import QueueManager

    import dask import distributed

    fractal_client = FractalClient("localhost:7777")
    workflow_client = distributed.Client("tcp://10.0.1.40:8786")

    ncores = 4
    memory_per_task = 2

    # Build a manager
    manager = QueueManager(fractal_client, workflow_client, cores_per_task=ncores, memory_per_task=mem)

    # Important for a calm shutdown
    from qcfractal.cli.cli_utils import install_signal_handlers
    install_signal_handlers(manager.loop, manager.stop)

    # Start or test the loop. Swap with the .test() and .start() method respectively
    manager.start()

Testing
-------

A ``qcfractal-manager` can be tested using the ``--test`` argument and does
not require an active ``qcfractal-manager``, this is very useful to check if
both the distributed workflow manager is setup correctly and correct
computational engines are found.

.. code-block:: console

    $ qcfractal-manager --test executor
    [I 190301 10:55:57 managers:118] QueueManager:
    [I 190301 10:55:57 managers:119]     Version:         v0.5.0+52.g6eab46f

    [I 190301 10:55:57 managers:122]     Name Information:
    [I 190301 10:55:57 managers:123]         Cluster:     unknown
    [I 190301 10:55:57 managers:124]         Hostname:    Daniels-MacBook-Pro.local
    [I 190301 10:55:57 managers:125]         UUID:        0cd257a6-c839-4743-bb33-fa55bebac1e1

    [I 190301 10:55:57 managers:127]     Queue Adapter:
    [I 190301 10:55:57 managers:128]         <ExecutorAdapter client=<ProcessPoolExecutor max_workers=8>>

    [I 190301 10:55:57 managers:131]     QCEngine:
    [I 190301 10:55:57 managers:132]         Version:    v0.6.1

    [I 190301 10:55:57 managers:158]     QCFractal server information:
    [I 190301 10:55:57 managers:159]         Not connected, some actions will not be available
    [I 190301 10:55:57 managers:389] Testing requested, generating tasks
    [I 190301 10:55:57 managers:425] Found program rdkit, adding to testing queue.
    [I 190301 10:55:57 managers:425] Found program torchani, adding to testing queue.
    [I 190301 10:55:57 managers:425] Found program psi4, adding to testing queue.
    [I 190301 10:55:57 base_adapter:124] Adapter: Task submitted rdkit
    [I 190301 10:55:57 base_adapter:124] Adapter: Task submitted torchani
    [I 190301 10:55:57 base_adapter:124] Adapter: Task submitted psi4
    [I 190301 10:55:57 managers:440] Testing tasks submitting, awaiting results.

    [I 190301 10:56:04 managers:444] Testing results acquired.
    [I 190301 10:56:04 managers:451] All tasks retrieved successfully.
    [I 190301 10:56:04 managers:456]   rdkit - PASSED
    [I 190301 10:56:04 managers:456]   torchani - PASSED
    [I 190301 10:56:04 managers:456]   psi4 - PASSED
    [I 190301 10:56:04 managers:465] All tasks completed successfully!



