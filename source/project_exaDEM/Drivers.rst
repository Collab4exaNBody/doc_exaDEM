Drivers
=======

This section provides an overview of the ``drivers`` implemented in ``ExaDEM``, followed by a detailed description of each ``driver`` integrated into the system.

Quick Overview of Drivers
^^^^^^^^^^^^^^^^^^^^^^^^^

What is a driver in ExaDEM
--------------------------

In ``ExaDEM``, a ``driver`` denotes a significant shape, such as walls, cylinders, or other geometric forms, which are treated differently from particles due to their unique characteristics. While the ExaDEM framework has been optimized and parallelized to effectively handle particles and polyhedra of comparable scales, ``drivers`` necessitate special treatment owing to their distinct properties. To accommodate multiple ``drivers`` in a simulation, ExaDEM offers flexibility through the creation of a ``drivers`` list. This feature allows users to include any number of ``drivers`` required to define boundary conditions and obstacles within the simulation environment.


The current implementation of ``ExaDEM`` includes a variety of ``drivers``, each serving specific purposes within the simulation framework. The detailed list of implemented ``drivers`` is provided in the following table:


.. list-table:: Glossary Of Drivers
   :widths: 25 25 25
   :header-rows: 1

   * - Name         
     - Driver Type 
     - Operator Name
   * - Cylinder
     - CYLINDER 
     - add_cylinder
   * - Wall / Surface 
     - SURFACE 
     - add_ball
   * - Ball / Sphere  
     - BALL       
     - add_surface
   * - STL Mesh 
     - STL_Mesh 
     - add_stl_mesh
   * - Undefined
     - UNDEFINED 
     - no operator

.. note::
 When adding a ``driver`` to the simulation in ``ExaDEM``, it is essential to define a Hooke parameter list specific to the ``driver`` within the `compute_hooke_interaction` operator.


Add a Driver To Your Simulation
-------------------------------

In ``ExaDEM``, ``drivers`` are managed differently depending on whether spheres or polyhedra are used in the simulation. Forces are computed per interaction for polyhedra, while forces are computed per sphere:

  * Using spheres, a special contact force is added to handle interactions with drivers in the ``hooke_force_driver`` operator.
  * Using polyhedra, special interactions (described in the Polyhedra section) are added to the interaction lists. Additionally, you need to specify your driver list in the list of operators called ``setup_drivers``, which is integrated into the default ``ExaDEM`` execution. It's crucial to specify an ID for each driver. If you create a second driver with an already used ID, it will overwrite the previous driver configuration.


Driver Descriptions
^^^^^^^^^^^^^^^^^^^

In this section, we provide brief descriptions of available ``drivers``. Please note that a test case is defined for each ``driver`` in the ``example`` directory.

Rotattion Drum / Cylinder
-------------------------

The rotation drum or cylinder driver represents an infinite cylinder along a specified axis. It is defined by parameters including its center, velocity, axis, and angular velocity.

.. |ex1end| image:: ../_static/rotating_drum_end.png
   :width: 300pt

|ex1end|

* Operator name: ``add_cylinder``
* Description: This operator add a cylinder to the drivers list.
* Parameters:

  * *id*: Driver index
  * *radius*: Cylinder radius
  * *axis*: Define the plan of the cylinder
  * *velocity*: Cylinder velocity, default is [0,0,0]
  * *angular_velocity*: Angular velocity of the cylinder, default is 0 m.s-1

YAML example:

.. code:: yaml

  - add_cylinder:
     id: 0
     center: [2.5, 2.5, 2.5]
     axis: [1, 0, 1]
     radius: 4
     angular_velocity: [0,0,0]

Wall / Surface
--------------

The wall or surface driver represents an infinite wall within the simulation environment. It is defined by parameters including its normal vector, offset, and velocity. Please note that currently, no angular velocity is defined for this driver. 

.. |ex4end| image:: ../_static/rigid_surface_end.png
   :width: 300pt

|ex4end|

* Operator name: ``add_surface``
* Description: This operator add a surface/wall to the drivers list.
* Parameters:

  * *id*: Driver index
  * *normal*: Normal vector of the rigid surface
  * *offset*: ffset from the origin (0,0,0) of the rigid surface
  * *velocity*: Wall/Surface velocity, default is [0,0,0]

YAML example:

.. code:: yaml

  - add_surface:
     id: 0
     normal: [0,0,1]
     offset: -0.5

Ball / Sphere
--------------

The ball or sphere driver represents a spherical object within the simulation environment. It is defined by parameters including its center, velocity, and angular velocity. This driver can be utilized as a boundary condition or obstacle in the simulation.

.. |ex3pend| image:: ../_static/ExaDEM/polyhedra_ball_end.png
   :width: 250pt

|ex3pend|

* Operator name: ``add_ball``
* Description: This operator add a ball / sphere (boundary condition or obstacle) to the drivers list.
* Parameters:

  * *center*: Center of the ball / sphere
  * *radius*: Radius of the ball / sphere
  * *velocity*: Velocity of the ball / sphere
  * *angular_velocity*: Angular velocity of the ball, default is 0 m.s-1

YAML example:

.. code:: yaml

  - add_ball:
     id: 0
     center: [2, 2, 0]
     radius: 20

STL Mesh
--------

The STL Mesh driver is constructed from an STL (Stereolithography) file to create a mesh of faces. This approach enables the rapid setup of complex geometries within the simulation environment. It's important to note that faces in an STL mesh are processed as a sphere polyhedron, meaning a small layer is added around each face.

.. |ex4pendmixte| image:: ../_static/ExaDEM/stl_mixte_end.png
   :width: 250pt

|ex4pendmixte|

* Operator name: ``add_stl_mesh``    
* Description: This operator add a stl mesh to the drivers list.
* Parameters:

  * *id*: Driver index
  * *filename*: Input filename (.stl)
  * *minskowski*: Minskowski radius value
  * *center*: Center is defined but not used
  * *velocity* : Velocity is defined but not used
  * *angular_velocity*: Angular_velocity is defined but not used

* Operator name: ``update_grid_stl_mesh``
* Description: Update the grid of lists of {vertices / edges / faces} in contact for every cells. The aim is to predefine a list of possible contacts with a cell for a stl mesh. These lists must be updated each time the grid changes. 
* Parameters: No parameter

YAML example:

.. code:: yaml

  - add_stl_mesh:
     id: 0
     filename: box_for_octa.stl
     minskowski: 0.01
  - update_grid_stl_mesh
