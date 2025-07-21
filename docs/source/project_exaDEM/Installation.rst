Installing ExaDEM
=================

``ExaDEM`` provides flexible installation methods to meet various user and developer needs. For user usage, the ``Spack`` package manager offers a straightforward installation process, ideal for user-only. 
However, for those who intend to develop or customize ``ExaDEM``, ``CMake`` is recommended, as it supports both `CPU` and `GPU` configurations (`GPU` support is not available with ``Spack``).

Choose the method below that best suits your setup and follow the instructions for a smooth installation experience.

Installation With CMake
^^^^^^^^^^^^^^^^^^^^^^^

Minimal Requirements
--------------------

The first step involves the installation of ``yaml-cpp``, which can be achieved using either the ``spack`` package manager or ``cmake``:


.. tabs::

   .. tab:: spack

      .. code-block:: bash

         spack install yaml-cpp@0.6.3
         spack load yaml-cpp@0.6.3

   .. tab:: apt-get install

      .. code-block:: bash

         sudo apt install libyaml-cpp-dev

   .. tab:: cmake 

      .. code-block:: bash

         export CURRENT_HOME=$PWD
         git clone --depth 1 --branch  yaml-cpp-0.6.3 https://github.com/jbeder/yaml-cpp.git
         mkdir ${CURRENT_HOME}/build-yaml-cpp && cd ${CURRENT_HOME}/build-yaml-cpp 
         cmake ../yaml-cpp/ -DCMAKE_BUILD_TYPE=Release -DCMAKE_INSTALL_PREFIX=${CURRENT_HOME}/install-yaml-cpp -DYAML_BUILD_SHARED_LIBS=ON -DYAML_CPP_BUILD_TESTS=OFF
         make install -j 4
         cd ${CURRENT_HOME}
         export PATH_TO_YAML=$PWD/install-yaml-cpp

Please ensure to remove `yaml-cpp` and `build-yaml-cpp`. When installing ``Onika``, ``exaNBody``, and ``exaDEM``, remember to add the following to your cmake command: ``-DCMAKE_PREFIX_PATH=${PATH_TO_YAML}``.

To proceed with the installation, your system must meet the minimum prerequisites (``MPI``, ``Onika``, and ``ExaNBody``). The next step involves the installation of ``Onika`` and ``ExaNBody``:

.. tabs::

   .. tab:: Ubuntu CPU

      .. code-block:: bash

         export CURRENT_HOME=$PWD
         git clone --branch v1.0.4 https://github.com/Collab4exaNBody/onika.git
         git clone --branch v2.0.4 https://github.com/Collab4exaNBody/exaNBody.git
         mkdir ${CURRENT_HOME}/build-onika && cd ${CURRENT_HOME}/build-onika
         cmake ${CURRENT_HOME}/onika -DCMAKE_BUILD_TYPE=Release -DCMAKE_INSTALL_PREFIX=${CURRENT_HOME}/install-onika -DONIKA_BUILD_CUDA=OFF
         make install -j 10
         export onika_DIR=${CURRENT_HOME}/install-onika
         mkdir ${CURRENT_HOME}/build-exanb && cd ${CURRENT_HOME}/build-exanb
         cmake ${CURRENT_HOME}/exaNBody -DCMAKE_BUILD_TYPE=Release -DCMAKE_INSTALL_PREFIX=${CURRENT_HOME}/install-exanb
         make install -j 10
         export exaNBody_DIR=${CURRENT_HOME}/install-exanb
         rm -rf ${CURRENT_HOME}/build-onika
         rm -rf ${CURRENT_HOME}/build-exanb

   .. tab:: Ubuntu GPU

      Please, select the correct compute capability for your ``GPU`` for ``DCMAKE_CUDA_ARCHITECTURES`` instead of 86 in this example.

      .. code-block:: bash

         export CURRENT_HOME=$PWD
         git clone --branch v1.0.4 https://github.com/Collab4exaNBody/onika.git
         git clone --branch v2.0.4 https://github.com/Collab4exaNBody/exaNBody.git
         mkdir ${CURRENT_HOME}/build-onika && cd ${CURRENT_HOME}/build-onika
         cmake ${CURRENT_HOME}/onika -DCMAKE_BUILD_TYPE=Release -DCMAKE_INSTALL_PREFIX=${CURRENT_HOME}/install-onika -DONIKA_BUILD_CUDA=ON -DCMAKE_CUDA_ARCHITECTURES=86
         make install -j 10
         export onika_DIR=${CURRENT_HOME}/install-onika
         mkdir ${CURRENT_HOME}/build-exanb && cd ${CURRENT_HOME}/build-exanb
         cmake ${CURRENT_HOME}/exaNBody -DCMAKE_BUILD_TYPE=Release -DCMAKE_INSTALL_PREFIX=${CURRENT_HOME}/install-exanb
         make install -j 10
         export exaNBody_DIR=${CURRENT_HOME}/install-exanb
         rm -rf ${CURRENT_HOME}/build-onika
         rm -rf ${CURRENT_HOME}/build-exanb  

   .. tab:: TOPAZE GPU

      Please, note that you need copy on topaze onika and exaNBody repository. You can load the ``YAML`` module such as ``module load yaml-cpp/`` and add this option to the ``cmake`` command: ``-DCMAKE_PREFIX_PATH=/ccc/products/yaml-cpp-0.6.3/system/default/``

      .. code-block:: bash

         module load gnu/13.2.0 cuda/12.4 mpi/openmpi/4.1.6 cmake/3.29.6
         cd $CCCSCRATCHDIR
         export CURRENT_HOME=$PWD
         // copy onika v1.0.4
         // copy exaNBody v2.0.4
         mkdir ${CURRENT_HOME}/build-onika && cd ${CURRENT_HOME}/build-onika
         cmake ${CURRENT_HOME}/onika -DCMAKE_BUILD_TYPE=Release -DCMAKE_INSTALL_PREFIX=${CURRENT_HOME}/install-onika -DONIKA_BUILD_CUDA=ON -DCMAKE_CUDA_ARCHITECTURES=80
         make install -j 10
         export onika_DIR=${CURRENT_HOME}/install-onika
         mkdir ${CURRENT_HOME}/build-exanb && cd ${CURRENT_HOME}/build-exanb
         cmake ${CURRENT_HOME}/exaNBody -DCMAKE_BUILD_TYPE=Release -DCMAKE_INSTALL_PREFIX=${CURRENT_HOME}/install-exanb
         make install -j 10
         export exaNBody_DIR=${CURRENT_HOME}/install-exanb
         rm -rf ${CURRENT_HOME}/build-onika
         rm -rf ${CURRENT_HOME}/build-exanb          


Optional Dependencies
---------------------

Before proceeding further, you have the option to consider the following dependencies:

- ``CUDA``
- ``HIP``
- ``RSA_MPI``

ExaDEM Installation
-------------------

To install ``ExaDEM``, follow these steps:

Set the exaNBody_DIR environment variable to the installation path. Clone the ``ExaDEM`` repository using the command:

.. code-block:: bash
		
   git clone https://github.com/Collab4exaNBody/exaDEM.git

Create a directory named build-exaDEM and navigate into it:

.. code-block:: bash
		
   mkdir build-exaDEM && cd build-exaDEM

Run CMake to configure the ExaDEM build:

.. tabs::

   .. tab:: cmake Minimal

      .. code-block:: bash
		
         cmake ../exaDEM -DCMAKE_BUILD_TYPE=Release 
         make -j 4
         source bin/setup-env.sh

   .. tab:: cmake GPU (a100)

      The gpu installation depends on onika.

      .. code-block:: bash

         cmake ../exaDEM -DCMAKE_BUILD_TYPE=Release 
         make -j 4
         source bin/setup-env.sh

   .. tab:: Spack

      See the spack section if you need to install spack.
 
      The ``spack_repo`` directory is in the exaDEM repository, you need to ``git clone`` exaDEM.

      .. code-block:: bash

         spack repo add spack_repo
         spack install exadem

.. warning::

  Old version (<= 1.1.1) It's important to note that the maximum number of vertices per particle shape is set to 8 by default. To change this value, you can specify this number by adding: ``-DEXADEM_MAX_VERTICES=N``.

.. warning::

    Please, do not forget to load the ``exaDEM`` environnement before running a job: `source bin/setup-env.sh`.

This command will display all plugins and related operators. Example: 

.. code-block:: bash
		
 + exadem_force_fieldPlugin
   operator    cylinder_wall
   operator    gravity_force
   operator    contact_force
   operator    rigid_surface
 + exadem_ioPlugin
   operator    print_simulation_state
   operator    read_xyz
   operator    read_dump_particles


Here are a few examples on ``CEA`` supercomputers:

.. tabs::

   .. tab:: CCRT Topaze Milan

      You need to specify "export exaNBody_DIR=${path_to_exaNBody}" and "export onika_DIR=${path_to_onika}"

      .. code-block:: bash

         module load yaml-cpp/0.6.3 gnu/13.2.0 mpi/openmpi/4.1.6 
         cmake ${path_to_exaDEM} -DCMAKE_BUILD_TYPE=Release 

   .. tab:: CCRT Topaze a100

      You need to specify "export exaNBody_DIR=${path_to_exaNBody}" and "export onika_DIR=${path_to_onika}"

      .. code-block:: bash

         module load gnu/13.2.0 cuda/12.4 mpi/openmpi/4.1.6 cmake/3.29.6
         cmake ${path_to_exaDEM} -DCMAKE_BUILD_TYPE=Release 

   .. tab:: CCRT Irene Skylake and Rome

      You need to specify "export exaNBody_DIR=${path_to_exaNBody}" and "export onika_DIR=${path_to_onika}"

      .. code-block:: bash

         module load yaml-cpp/0.6.3 gnu/13.2.0 mpi/openmpi/4.1.6
         cmake ${path_to_exaDEM} -DCMAKE_BUILD_TYPE=Release 

Launch examples / ctest
-----------------------

A set of minimal test cases can be run using the following command (non-regression test) in your `build` repository:

.. code-block:: bash

  ctest --test-dir example

You can also add exaDEM to your bashrc by adding an alias (please, replace YOURDIR by your build directory): 

.. code-block:: bash

  vi ~/.bashrc
  alias exaDEM='~/YOURDIR/build/exaDEM'

Or just on your local environment:

  alias exaDEM=$PWD/exaDEM

Installation With Spack
^^^^^^^^^^^^^^^^^^^^^^^
Installation with ``spack`` is preferable for people who don't want to develop in ``exaDEM``. Only stable versions are added when you install ``ExaDEM`` with ``Spack``.

.. note::
  The main of ``ExaDEM`` will never be directly accessible via this installation method.

Installing Spack
----------------

.. code-block:: bash

  git clone --depth=2 --branch=v0.23.0 https://github.com/spack/spack.git
  export SPACK_ROOT=$PWD/spack
  source ${SPACK_ROOT}/share/spack/setup-env.sh

Installing ExaDEM
-----------------

First get the ``spack`` repository in exaDEM directory and add it to spack. It contains two packages: ``exanbody`` and ``exadem``:

.. code-block:: bash
		
   git clone https://github.com/Collab4exaNBody/spack-repos.git
   spack repo add spack-repos


Second install ``ExaDEM`` (this command will install ``cmake``, ``yaml-cpp``, ``onika`` and ``exanbody``).

.. code-block:: bash

  spack install exadem

Running your simulation
^^^^^^^^^^^^^^^^^^^^^^^

CMake
-----

Now that you have installed the ``ExaDEM`` and ``exaNBody`` packages, you can create your simulation file in ``YAML`` format (refer to the 'example' folder or the documentation for each operator). Once this file is constructed, you can initiate your simulation using the following instructions.

.. code-block:: bash
		
   export N_OMP=1
   export N_MPI=1
   export OMP_NUM_THREADS=$N_OMP
   mpirun -n $N_MPI ./exaDEM test-case.msp

Spack
-----

The ``ExaDEM`` executable has been created in the spack directory. You can run your simulation with your input file (*your_input_file.msp*) such as:

.. code-block:: bash

  spack load exadem
  exaDEM your_input_file.msp


Adding Optional Packages:
^^^^^^^^^^^^^^^^^^^^^^^^^

RSA
---

This external library is available here: ``https://github.com/MarcJos/RSA_MPI``

You can install it with the following commands:

.. code-block:: bash

  export rsa_mpi_DIR=$DIR
  cmake ../exaDEM -DUSE_RSA=ON
