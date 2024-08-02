Installation
============

Installation With CMAKE
^^^^^^^^^^^^^^^^^^^^^^^

Minimal Requirements
--------------------

To proceed with the installation, your system must meet the minimum prerequisites. The first step involves the installation of ``exaNBody``:

.. code-block:: bash

   git clone https://github.com/Collab4exaNBody/exaNBody.git
   export exaNBody_DIR=${PWD}/exaNBody 

The next step involves the installation of ``yaml-cpp``, which can be achieved using either the ``spack`` package manager or ``cmake``:

.. code-block:: bash

   spack install yaml-cpp@0.6.3
   spack load yaml-cpp@0.6.3

Optional Dependencies
---------------------

Before proceeding further, you have the option to consider the following dependencies:

- ``CUDA``
- ``HIP``
- ``MPI``

ExaDEM Installation
-------------------

To install ``ExaDEM``, follow these steps:

Set the exaNBody_DIR environment variable to the installation path. Clone the ``ExaDEM`` repository using the command:

.. code-block:: bash
		
   git clone https://github.com/Collab4exaNBody/exaDEM.git

Create a directory named build-exaDEM and navigate into it:

.. code-block:: bash
		
   mkdir build-exaDEM && cd build-exaDEM

Run CMake to configure the ExaDEM build, specifying that ``CUDA`` support should be turned off:

.. code-block:: bash
		
   cmake ../exaDEM -DXNB_BUILD_CUDA=OFF

Build ``ExaDEM`` using the make command with a specified number of parallel jobs (e.g., -j 4 for 4 parallel jobs):

.. code-block:: bash
		
   make -j 4

Build Plugins

.. code-block:: bash
		
   make UpdatePluginDataBase

This command will display all plugins and related operators. Example: 

.. code-block:: bash
		
 + exadem_force_fieldPlugin
   operator    cylinder_wall
   operator    gravity_force
   operator    hooke_force
   operator    rigid_surface
 + exadem_ioPlugin
   operator    print_simulation_state
   operator    read_xyz
   operator    read_dump_particles

Running your simulation
-----------------------

Now that you have installed the ``ExaDEM`` and ``exaNBody`` packages, you can create your simulation file in ``YAML`` format (refer to the 'example' folder or the documentation for each operator). Once this file is constructed, you can initiate your simulation using the following instructions.

.. code-block:: bash
		
   export N_OMP=1
   export N_MPI=1
   export OMP_NUM_THREADS=$N_OMP
   mpirun -n $N_MPI ./exaDEM test-case.msp


Installation With Spack
^^^^^^^^^^^^^^^^^^^^^^^
Installation with ``spack`` is preferable for people who don't want to develop in ``exaDEM``. Only stable versions are added when you install ``ExaDEM`` with ``Spack``.

.. note::
  The main of ``ExaDEM`` will never be directly accessible via this installation method.

Installing Spack
----------------

.. code-block:: bash

  git clone https://github.com/spack/spack.git
  export SPACK_ROOT=$PWD/spack
  source ${SPACK_ROOT}/share/spack/setup-env.sh

Installing ExaDEM
-----------------

First get the ``spack`` repository in exaDEM directory and it to spack. It contains two packages: ``exanbody`` and ``exadem``:

.. code-block:: bash
		
   git clone https://github.com/Collab4exaNBody/exaDEM.git
   cd exaDEM
   spack repo add spack_repo

.. note::
  
  Current variante(s):
  
  * +cuda: Add GPU support

Second install ``ExaDEM`` (this command will install ``cmake``, ``yaml-cpp`` and ``exanbody``).

.. code-block:: bash

  spack install exadem

Finally the ``ExaDEM`` executable has been created in the spack directory. You can run your simulation with your input file (*your_input_file.msp*) such as:

.. code-block:: bash

  spack load exadem
  exaDEM your_input_file.msp
