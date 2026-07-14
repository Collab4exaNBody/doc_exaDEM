================
ExaDEM Changelog
================

All notable changes to this project will be documented in this file.


Release Note
^^^^^^^^^^^^

Release Notes v1.2.3 (in development)
--------------------------------------

.. note::

   This release is under development on branch ``349-integrate-group-field`` and has not been tagged yet.

New Features:

   * Fields

      - Add a ``group`` particle field, used to select multi-material contact parameters independently from the particle ``type``. Several particle types can share the same group and therefore reuse the same contact law.
      - Add the ``group`` parameter to ``set_fields``.
      - Add the ``set_group`` operator to (re)assign the ``group`` field to particles according to their type.

   * I/O

      - Add support for the ``group`` field in dump file readers/writers, including legacy Rockable readers (legacy writers are not affected).

Changes and Enhancements:

   * Force Field

      - ``multimat_contact_params``, ``drivers_contact_params`` and ``inner_bond_params`` now select contact parameters using ``group1``/``group2`` (or ``group``) instead of particle type names (``mat1``/``mat2``/``mat``). See the Multi-Material section of the Force Field page.

Release Notes v1.2.2 (07/26)
-----------------------------

New Features:

   * Fragmentation / Inner Bonds

      - Add operator to randomly change sphere radius
      - Add inner bonds and driver support to compute the stress per particle
      - Add criterion mode (mixed and separated)
      - Add ``dump_broken_interface`` and a ParaView post-processing script
      - Add stress energy criterion
      - Add inner bonds in the dump contact network

   * Visualization / ParaView

      - Improve ``ParaviewNetwork`` and the criterion used when writing interactions
      - Add cluster field for the ``paraview_polyhedra`` operator
      - Add ParaView output for cylinders
      - Add ParaView export in ``pvtu`` format

   * Drivers

      - Rework ``driver_params`` and ``MotionType`` in the driver plugin
      - Use a dt scale in driver numerical schemes
      - Add a ``driverextractor``
      - Fix driver surface corrections on GPU

Changes and Enhancements:

   * Performance

      - Integrate the interaction list build step
      - Optimize ``unclassify`` with binary search
      - Reduce compilation time for contact polyhedron/sphere
      - Use a dedicated extra storage for ``rshape`` data
      - Use scratch memory for ``nbh_polyhedron_gpu``

   * Cleanup and Refactoring

      - Clean the drivers plugin
      - Move ``classify``/``unclassify`` implementation to ``.cpp`` files
      - Move interaction manager implementation to ``.cpp`` files
      - Rename ``interaction_buffers`` to ``contact_state``
      - Clean the shape plugin
      - New convention for structure members: ``name`` -> ``name_``

   * CI / Build

      - Add a CUDA CI pipeline
      - Use CUDA 13 in CUDA CI

Bug Fixes:

   - Fix ghost interactions handling in ``nbh_polyhedron_gpu``
   - Fix ill-formed interactions in ``nbh`` operators
   - Fix compilation in debug mode
   - Update ``grid_cell_values`` on ``ghost_update``
   - Fix use of MEM PREFETCH with CUDA 13
   - Fix logs and add a face shape identification tolerance
   - Fix periodic boundary conditions with ``DOMAIN_BOUNDS`` mode (rockable)

Release Notes v1.2.1 (03/26)
-----------------------------

New Features:

   - Add a ``deposit`` plugin.
   - Add an operator to add particles from an XYZ file.

Changes and Enhancements:

   * I/O

      - Switch from ``grid_vtklegacy`` to ``grid_vtk`` for grid output.

   * Fragmentation

      - Add support for weights in inner bond force computations.

   * General

      - Add an additional validation check in ``check_rcut``.

Release Notes v1.2.0 (02/26)
-----------------------------

New Features:

   * Fragmentation System

      - Add inner bond interaction (rework of the interaction system)
      - Add interfaces (set of inner bond interactions)
      - Add a routine to stick polyhedra
      - Add field ``cluster``
      - Add ``config_fragmentation.msp``

   * Shapes

      - Add a default shapes library (``cube``, ``sphere``, ``rice`` grain)

   * Drivers

      - Add a deformation operator for STL meshes
      - Rename/refactor ``Stl_mesh`` to ``RShapeDriver``
      - Refactor ``FORCE_MOTION`` into a more versatile ``PARTICLE`` motion type
      - Add a "Freeze Particles" operator
      - Improve expression-based motion types to account for angular moments

Changes and Enhancements:

   * Performance

      - Copy interactions in the ghost area (MPI)
      - Compute the contact interface criterion on GPU
      - Update ``BackupDEM`` structures for more efficient checkpointing

   * Post-Processing

      - Add a traction/shear test case for fragmentation validation
      - Improve ParaView outputs for particle interfaces
      - Fix vertex ordering in interface exports

Bug Fixes:

   - Fix compatibility with ExaNBody 2.0.7
   - Fix region usage in ``set_fields``
   - Improve radius-from-shape detection logic

Release Notes v1.1.6 (12/25)
-----------------------------

New Features:

   - Add TinyExpr support and projected quantities
   - Add a motion modification operator
   - Enable STL driver control using moments
   - Add a script to visualize driver-applied stresses by component
   - Improve ParaView output for contact networks
   - Add a DMT enumeration

Bug Fixes:

   - Fix multiple Rockable behavior issues
   - Fix ParaView export bugs for polyhedra and vertex displacement
   - Fix simulation box definition and the Rockable dump writer
   - Fix an MPI bug in linear compressible motion
   - Add validation for a missing ``preCompDone`` flag in shape files

Release Notes v1.1.5 (10/25)
-----------------------------

Bug Fixes:

   - Fix ``linear_compressive_forces`` and ``linear_motion`` modes when moving a surface in the negative direction.

Added Examples:

   - Spheres with linear motion types
   - Polyhedra with linear compressive force

Release Notes v1.1.4 (09/25)
-----------------------------

New Features:

   - Add a pendulum motion type for the Surface driver
   - Add the ``check_consistency_grid_classifier`` operator
   - Add a rescale Minkowski option in ``read_shapes``
   - Implement ``exadem-config`` file generation

Changes and Enhancements:

   * Examples

      - Update the axial-stress example for spheres
      - Update rotating-drum examples
      - Refactor ``polyhedra/rotating_drum``, ``polyhedra/stl_mesh`` and ``polyhedra/balls``
      - Refactor ``ball/driver`` examples and fix neighbor list update
      - Refactor the ``spheres/movable-wall`` example
      - Update and factorize analysis examples
      - Remove deprecated examples under ``depot``

   * Regression Tests

      - Add regression tests for ``sphere/cylinder_stl``, mirror boundary conditions, ``polyhedra/defbox``, ``polyhedra/stl_cylinder``, ``spheres/piston`` and funnel

Release Notes v1.1.3 (09/25)
-----------------------------

New Features:

   - Add a mixer simulation example
   - Add examples with large vertex sets
   - Integrate OBBTree (from Rockable) for improved collision detection
   - Add shape rescaling at import time

Changes and Enhancements:

   - Improve required-slot handling
   - Centralize and factorize error management (``color_log::error``, ``color_log::warning``)

Bug Fixes:

   - Fix ParaView export of STL meshes
   - Fix MSP driver output files

Release Notes v1.1.2 (07/25)
-----------------------------

New Features:

   - Add support for dumping Rockable object output files
   - Add a new internal vertex data layout for improved efficiency
   - Add customizable options to ``compute_cell_particles``
   - Add a Shaker motion type for STL mesh drivers (sinusoidal displacement)

Bug Fixes:

   - Fix ParaView shape data output

Release Notes v1.1.1 (05/25)
-----------------------------

New Features:

   - Add the ``check_rcut`` operator to verify the interaction cutoff radius.
   - Add a multi-material contact law supporting multiple parameter sets.
   - Add an RSA-based particle generator targeting a given volume fraction.
   - Add a slot to override domain bounds when importing geometry with ``read_xyz``.
   - Add logging for the critical time step.

Release Notes v1.1.0 (03/25)
----------------------------

New Features:

   * Fields
   
      - Add Unified operator for initializing fields: ``set_fields``

   * Drivers

       * Add driver motion type:
        
         - ``STATIONARY``
         - ``LINEAR_MOTION``
         - ``COMPRESSIVE_FORCE``
         - ``LINEAR_FORCE_MOTION``
         - ``FORCE_MOTION``
         - ``LINEAR_COMPRESSIVE_MOTION``
         - ``TABULATED``
         
      * Add a reader for binary stl files
   
   * Analysis

      * Add new analyses:

         - Compute barycenters per type 
         - Count the number of particles per type 

Changes and Enhancements:

   * I/O

      - Optimized the contact network (add information)

   * Update deformation domain: available for spheres and polyhedra

Release Notes v1.0.2 (11/24)
----------------------------

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

        - Introduced a classifier to categorize interactions and enhance data organization.
        - Provides improved management and processing of interaction data.

   * Mirror Boundary Conditions from ExaNBody Are Available

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
