.. image:: ../_static/exaDEMlogo2.png
   :scale: 100 %
   :align: center

ExaDEM Software
===============

Overview of ExaDEM
^^^^^^^^^^^^^^^^^^

``ExaDEM`` is a software solution in the field of computational simulations. It's a Discrete Element Method (``DEM``) code developed within the ``exaNBody framework``. This framework provides the basis for DEM functionalities and performance optimizations. A notable aspect of ``ExaDEM`` is its hybrid parallelization approach, which combines the use of ``MPI`` (Message Passing Interface) and threads (``OpenMP``). This combination aims to enhance computation times for simulations, making them more efficient and manageable.

Additionally, ``ExaDEM`` offers compatibility with ``MPI``+``GPUs``, using the ``CUDA`` programming model (Onika layer). This feature provides the option to leverage ``GPU`` processing power for potential performance gains in simulations. Written in ``C++17``, ``ExaDEM`` is built on a contemporary codebase. It aims to provide researchers and engineers with a tool for addressing ``DEM`` simulations.

ExaDEM Capabilities
^^^^^^^^^^^^^^^^^^^

The following table provides a glossary of features supported by ``ExaDEM``, categorized by their computational platform (``CPU`` or ``GPU``) and particle type (Sphere or Polyhedron). Please note that this information is indicative and may be subject to change based on updates or modifications to the ``ExaDEM`` framework.

.. list-table:: Glossary Of Features
  :widths: 40 15 15 15 15
  :header-rows: 1

  * - Features
    - Sphere(CPU)
    - Sphere(GPU)
    - Polyhedron(CPU)
    - Polyhedron(GPU)
  * - Field Set Plugin
    - ✔
    - ✔
    - ✔
    - ✔
  * - I/O Files
    - ✔
    - ✘
    - ✔
    - ✘
  * - Paraview Files
    - ✔
    - ✘
    - ✔
    - ✘
  * - Particle Migration (MPI)
    - ✔
    - ✘
    - ✔
    - ✘
  * - Load Balancing (RCB)
    - ✔
    - ✘
    - ✔
    - ✘
  * - Numerical Scheme Plugin
    - ✔
    - ✔
    - ✔
    - ✔
  * - Regions
    - ✔
    - ✔
    - ✔
    - ✔
  * - Particles Generator
    - ✔
    - ✘
    - ✔
    - ✘
  * - Contact Law
    - ✔
    - ✔
    - ✔
    - ✔
  * - Gravity Force
    - ✔
    - ✔
    - ✔
    - ✔
  * - Quadratic Drag Force
    - ✔
    - ✔
    - ✔
    - ✔
  * - Driver Cylinder
    - ✔
    - ✔
    - ✔
    - ✔
  * - Driver Wall/Surface
    - ✔
    - ✔
    - ✔
    - ✔
  * - Driver type Ball/Sphere
    - ✔
    - ✔
    - ✔
    - ✔
  * - Driver type STL Mesh
    - ✔
    - ✔
    - ✔
    - ✔
  * - Contact Network File
    - ✔
    - ✘
    - ✔
    - ✘

.. note::

  With version 1.0.1, ``exaDEM`` can now run ``GPU`` calculations with polyhedra. Nevertheless, optimizations are required to achieve good performance. These are currently being implemented.


Default Execution Graph With ExaDEM
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

``ExaDEM`` is based on the operator execution system implemented by the ``ExaNBody`` framework. In particular, this allows us to define a set of default operators for ``ExaDEM`` to run a ``DEM`` simulation following a default Verlet Vitesse integration scheme. ``ExaDEM`` offers two variants, depending on whether you want to simulate spherical particles only or spheropolyhedron particles. To choose between these two variants, you'll need to include either *config_spheres.msp* or *config_polyhedra.msp* at the beginning of your input file. If neither is defined, the simulation will not work. Including these files allows you to override certain operators left at *nop* (for no operation). The graph below shows the sequence of ``exaDEM``'s main operators. 

.. image:: ../_static/ExaDEM/ExaDEM-DAG.png
   :scale: 100 %
   :align: center


.. note::
  - Violet: These operators are only called with the polyhedra variant.
  - Blue: Numerical time scheme operators.

Release Note
^^^^^^^^^^^^

Release Notes for Version 1.0.2 (11/24) [In Preparation]

New Features

   * Drivers

       - Added moving and rotating drivers, with special support for stl_mesh drivers.
       - Introduced timestep operators.
       - Implemented I/O driver operators.

   * I/O

       - Defined a tree structure for output files (io_config).
       - Computed system stress tensor.
       - Added example for writing to XYZ files.
       - Included post-processing scripts for easier data analysis.

Changes and Enhancements

   * Drivers

       - Optimized stl_mesh to efficiently build the lookup grid.

   * Classifier for Interactions

       - Improved interaction storage in the classifier (InteractionWrapper) using SOA (Structure of Arrays) data structures.

   * I/O

       - Added new logs to track active interactions and maximum interpenetration.

   * General

       - Renamed hooke_force operator to contact_force for clarity.
       - Limited browsing to cells containing particles only.

Bug Fixes

   - Fixed shape name paths to be relative.
   - Updated interaction statistics to print the number of interactions per driver.


Release Notes for Version 1.0.1 (06/24):
----------------------------------------

New Features:

   * Classifier for Interactions

        - Introduced a classifier to categorize interactions and enhancing data organization.
        - Provides improved management and processing of interaction data.

   * Mirror Boundary Conditions from ExaNBody Are Availables

        - Mirror boundary conditions to simulate reflections at domain boundaries.

   * GPU Version for Polyhedra

        - Added GPU-accelerated support for polyhedra computations.

Changes and Enhancements:

   * Unified Drivers for Spheres and Polyhedra

        - Integrated a unified driver system for both spherical and polyhedral objects.
        - Simplifies driver management and enhances code reusability.

Removed Features:

   * Meshset and Friction Plugins

        - Removed deprecated meshset and friction plugins.
        - Reduces complexity.

Added Examples:

   * New Example: Funnel
   * Mirror Boundary Conditions
