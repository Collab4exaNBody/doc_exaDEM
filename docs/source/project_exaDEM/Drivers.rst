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
 When adding a ``driver`` to the simulation in ``ExaDEM``, it is essential to define a Contact parameter list specific to the ``driver`` within the `compute_contact_interaction` operator.


Add a Driver To Your Simulation
-------------------------------

In ``ExaDEM``, ``drivers`` are managed differently depending on whether spheres or polyhedra are used in the simulation. Forces are computed per interaction for polyhedra, while forces are computed and summed per sphere body:

  * Using spheres, a special contact force is added to handle interactions with drivers in the ``contact_force_driver`` operator.
  * Using polyhedra, special interactions (described in the Polyhedra section) are added to the interaction lists. Additionally, you need to specify your driver list in the list of operators called ``setup_drivers``, which is integrated into the default ``ExaDEM`` execution. It's crucial to specify an ID for each driver. If you create a second driver with an already used ID, it will overwrite the previous driver configuration.


Driver Descriptions
^^^^^^^^^^^^^^^^^^^

In this section, we provide brief descriptions of available ``drivers``. Please note that a test case is defined for each ``driver`` in the ``example`` directory.

Rotating Drum / Cylinder
-------------------------

The rotating drum or cylinder driver represents an infinite cylinder rotating along a specified axis. It is defined by parameters including its center, velocity, axis, and angular velocity.

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
  * *center*: Center of the cylinder

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
  * *center*: Center of the surface (used for rotation when the angular velocity is defined)
  * *normal*: Normal vector of the rigid surface
  * *offset*: ffset from the origin (0,0,0) of the rigid surface
  * *velocity*: Wall/Surface velocity, default is [0,0,0]
  * *vrot*: Angular velocity of the surface, default is 0 m.s-1

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
  * *vrot*: Angular velocity of the ball, default is 0 m.s-1


YAML example:

.. code:: yaml

  - add_ball:
     id: 0
     center: [2, 2, 0]
     radius: 20

STL Mesh
--------

The STL Mesh driver is constructed from an .STL (Stereolithography) file to create a mesh of faces. This approach enables the rapid setup of complex geometries within the simulation environment. It's important to note that faces in an STL mesh are processed as a sphere polyhedron, meaning a small layer is added around each face.

.. |ex4pendmixte| image:: ../_static/ExaDEM/stl_mixte_end.png
   :width: 250pt

|ex4pendmixte|

* Operator name: ``add_stl_mesh``    
* Description: This operator add a stl mesh to the drivers list.
* Parameters:

  * *id*: Driver index
  * *filename*: Input filename (.stl or .shp)
  * *minskowski*: Minskowski radius value
  * *center*: Center is defined but not used
  * *velocity* : Velocity is defined but not used
  * *angular_velocity*: Angular_velocity of the mesh
  * *orientation*: Orientation of the mesh.

* Operator name: ``update_grid_stl_mesh``
* Description: Update the grid of lists of {vertices / edges / faces} in contact for every cells. The aim is to predefine a list of possible contacts with a cell for a stl mesh. These lists must be updated each time the grid changes. 
* Parameters: No parameter

YAML example:

.. code:: yaml

  - add_stl_mesh:
     id: 0
     filename: box_for_octa.stl
     minskowski: 0.01

I/O Drivers
^^^^^^^^^^^

An input/output system has been implemented primarily for drivers performing movements, such as a rigid surface compressing a sample or a blade rotating around an axis.


The drivers' output is automatically triggered when the user sets the global variable: ``simulation_dump_frequency``. This command also allows particles and interactions to be stored in a separate file. The drivers are then saved in a file located at ``ExaDEMOutputDir/CheckpointFiles/driver_%010d.msp``, containing the drivers' information. In the case of an ``STL mesh`` driver, a shp file is added to the ``ExaDEMOutputDir/CheckpointFiles/`` directory, which contains the geometry of the ``STL mesh``.To restart the driver along with your simulation, simply include the ``.msp`` file containing the ``setup_driver`` operator block at the beginning of your restart file.

YAML example: 

.. code:: yaml

  grid_flavor: grid_flavor_dem
  includes:
    - config_spheres.msp
    - ExaDEMOutputDir/CheckpointFiles/driver_0000040000.msp
  global:
    simulation_dump_frequency: 10000


Similarly, ExaDEM saves ``STL meshes`` each time a Paraview output is generated by setting the global variable: ``simulation_paraview_frequency``. The ``STL mesh`` is then translated and oriented correctly in the ``ExaDEMOutputDir/ParaviewOutputFiles/`` directory as ``shape_name_iteration.vtk``.

Advanced Operators
^^^^^^^^^^^^^^^^^^

Updage Grid For Stl Mesh
------------------------

The purpose of this operator is to project the stl mesh onto the cells making up the exaDEM grid in order to speed up the search for interactions. Each grid cell is then assigned a set of vertices, edges and faces that are potentially in contact with the cell's particles.

* Operator name: ``grid_stl_mesh``    
* Description: Update the list of information for each cell regarding the vertex, edge, and face indices in contact with the cell in an STL mesh.
* Parameters:
  - *force_reset*: Force to rebuild grid for stl meshes

.. note::

  [1] This operator only projects the stl mesh onto the grid making up the MPI process sub-domain. If the sub-domain changes, the update must be forced (force_reset=0). 
  [2] If the stl mesh is stationary (v= null, vrot=null), the grid is not updated. This speeds up calculations when the stl mesh has many elements.

YAML example: 

.. code:: yaml

  - compute_driver_vertices:
     force_reset: true

Compute Driver Vertices
-----------------------

This operator is used to update the vertex positions of operators with vertices. For the moment, this operator is only used for stl meshes and to fill in the `vertices` field.

* Operator name: ``compute_driver_vertices``    
* Description: This operator calculates new vertex positions.
* Parameters:
  - *force_host*: Force computations on the host

.. note::

  For GPU performance reasons, you may decide not to update the GPU data directly, knowing that it will be used to build the CPU interaction list.

YAML example: 

.. code:: yaml

  - compute_driver_vertices:
     force_host: false
