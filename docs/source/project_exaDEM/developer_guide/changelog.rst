================
ExaDEM Changelog
================

All notable changes to this project will be documented in this file.


Release Note
^^^^^^^^^^^^

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
