Test cases
==========

You can explore various basic test cases located in the `example` directory. These test cases serve as illustrative examples of ExaDEM's functionality and can assist you in understanding its behavior and capabilities.


Example Using Spheres
---------------------

Example 1: Rotating drum
^^^^^^^^^^^^^^^^^^^^^^^^

A DEM simulation of a rotating drum involves modeling the movement of spherical particles within a drum container as it rotates. Through this simulation, we can observe how particles interact, collide, and move in response to the drum's motion. This provides insights into phenomena like particle segregation, convection currents, and mixing patterns, contributing to improved understanding of granular material behavior in rotational scenarios.

.. |ex1start| image:: ../_static/rotating_drum_start.png
   :width: 300pt

.. |ex1end| image:: ../_static/rotating_drum_end.png
   :width: 300pt

+--------------------------+--------------------------+
| .. centered:: Rotating Drum                         |
+--------------------------+--------------------------+
| .. centered:: Start      | .. centered:: End        |
+==========================+==========================+
| |ex1start|               | |ex1end|                 |
+--------------------------+--------------------------+

Example 3: Rigid stress
^^^^^^^^^^^^^^^^^^^^^^^

In a DEM simulation under radial stress, particles are exposed to pressure from a central point, causing an outward force in all directions. Using the Discrete Element Method (DEM) to simulate this scenario allows us to analyze how particles within a system react to the applied radial stress. The simulation offers insights into particle rearrangements, contact forces, and structural changes, giving us a deeper understanding of granular material behavior under radial loading conditions.

.. |ex3start| image:: ../_static/radial_stress_start.png
   :width: 300pt

.. |ex3end| image:: ../_static/radial_stress_end.png
   :width: 300pt

+--------------------------+--------------------------+
| .. centered:: Radial Stress                         |
+--------------------------+--------------------------+
| .. centered:: Start      | .. centered:: End        |
+==========================+==========================+
| |ex3start|               | |ex3end|                 |
+--------------------------+--------------------------+

Example 4: Rigid surface
^^^^^^^^^^^^^^^^^^^^^^^^

A DEM simulation involving spherical particles falling onto a rigid surface offers a virtual exploration of particle dynamics in a gravity-driven scenario. This simulation captures the behavior of individual spherical particles as they descend and interact with a solid surface below.

.. |ex4start| image:: ../_static/rigid_surface_start.png
   :width: 300pt

.. |ex4end| image:: ../_static/rigid_surface_end.png
   :width: 300pt

+--------------------------+--------------------------+
| .. centered:: Rigid Surface                         |
+--------------------------+--------------------------+
| .. centered:: Start      | .. centered:: End        |
+==========================+==========================+
| |ex4start|               | |ex4end|                 |
+--------------------------+--------------------------+

Example 5: Impose Velocity
^^^^^^^^^^^^^^^^^^^^^^^^^^

In this DEM simulation, a scenario is simulated where a group of particles with imposed velocity occupies a defined area. As other particles fall into this region, they interact with the moving particles, impacting their trajectories. The simulation provides insights into how moving particles influence the behavior of surrounding particles.

.. |ex5start| image:: ../_static/impose_velocity_start.png
   :width: 300pt

.. |ex5end| image:: ../_static/impose_velocity_end.png
   :width: 300pt

+--------------------------+--------------------------+
| .. centered:: Impose Veclocity                      |
+--------------------------+--------------------------+
| .. centered:: Start      | .. centered:: End        |
+==========================+==========================+
| |ex5start|               | |ex5end|                 |
+--------------------------+--------------------------+

Example 6 : Movable wall
^^^^^^^^^^^^^^^^^^^^^^^^

In this DEM simulation, a cluster of spherical particles is constrained against a rigid surface. A piston is introduced to apply a steadily increasing stress that linearly evolves over time. This simulation captures the dynamics as the piston's force gradually grows. As the piston imparts its stress, the particle block undergoes deformation and stress propagation. 

.. |ex6start| image:: ../_static/movable_wall_start.png
   :width: 300pt

.. |ex6end| image:: ../_static/movable_wall_end.png
   :width: 300pt

+--------------------------+--------------------------+
| .. centered:: Movable Wall                          |
+--------------------------+--------------------------+
| .. centered:: Start      | .. centered:: End        |
+==========================+==========================+
| |ex6start|               | |ex6end|                 |
+--------------------------+--------------------------+

Example 7 : Using a STL Mesh
^^^^^^^^^^^^^^^^^^^^^^^^^^^^

In this DEM simulation, a cluster of spherical particles is  a cluster of particles falls onto an stl mesh and into a box. This case study highlights the use of meshes containing numerous facets.

.. |ex7start| image:: ../_static/mesh_stl_start.png
   :width: 300pt

.. |ex7end| image:: ../_static/mesh_stl_end.png
   :width: 300pt

+--------------------------+--------------------------+
| .. centered:: Mesh STL                              |
+--------------------------+--------------------------+
| .. centered:: Start      | .. centered:: End        |
+==========================+==========================+
| |ex7start|               | |ex7end|                 |
+--------------------------+--------------------------+

Example 9 : Particle Generation With RSA Algorithm
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

In this DEM simulation, a cluster of 287,642 spherical particles has been generated by the rsa algorithm. Then, particles fall by gravity in a drum.

.. |ex8start| image:: ../_static/rsa_start.png
   :width: 300pt

.. |ex8end| image:: ../_static/rsa_end.png
   :width: 300pt


+--------------------------+--------------------------+
| .. centered:: RSA                                   |
+--------------------------+--------------------------+
| .. centered:: Start      | .. centered:: End        |
+==========================+==========================+
| |ex8start|               | |ex8end|                 |
+--------------------------+--------------------------+

Example 10 : Jet
^^^^^^^^^^^^^^^^


This example demonstrates the application of a velocity field to spheres based on a Cartesian grid projection. Although it does not represent a physical scenario, a geyser-like effect has been simulated using a cylindrical shape, directing the particle velocities towards a specified speed. Future developments will involve applying non-uniform velocity fields to simulate more complex fluid configurations.

.. |ex10starthalf| image:: ../_static/ExaDEM/jet_half_start.png
   :width: 250pt

.. |ex10endhalf| image:: ../_static/ExaDEM/jet_half_end.png
   :width: 250pt


.. |ex10startfull| image:: ../_static/ExaDEM/jet_full_start.png
   :width: 250pt

.. |ex10endfull| image:: ../_static/ExaDEM/jet_full_end.png
   :width: 250pt

+--------------------------+--------------------------+
| .. centered:: Geyser Simulation                     |
+--------------------------+--------------------------+
| .. centered:: Start      | .. centered:: End        |
+==========================+==========================+
| |ex10starthalf|          | |ex10endhalf|            |
+--------------------------+--------------------------+
| |ex10startfull|          | |ex10endfull|            |
+--------------------------+--------------------------+

Example 11 : Mirror Boundary Conditions
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

This example tests the mirror conditions available in exaNBody. Although these conditions are not directly applicable (" not physics "), because all fields are copied identically (without processing/filtering) at each time step in the ghost cells (e.g. velocity, moments), except for positions (axial symmetry). This example highlights this functionality, and could potentially be coupled with other operators to develop new boundary conditions (e.g. resetting velocities to 0 to model a rigid surface). This example involves dropping 33120 spheres, adding mirror boundary conditions in all directions and letting them fall by gravity. 


.. |ex11start| image:: ../_static/ExaDEM/mirror_start.png
   :width: 250pt

.. |ex11end| image:: ../_static/ExaDEM/mirror_end.png
   :width: 250pt

+--------------------------+--------------------------+
| .. centered:: Mirror Simulation                     |
+--------------------------+--------------------------+
| .. centered:: Start      | .. centered:: End        |
+==========================+==========================+
| |ex11start|              | |ex11end|                |
+--------------------------+--------------------------+


Examples Using Polyhedra
------------------------

Example 1: Polyhedra Generation Frequency
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^


In this example, we simulate the generation of 100 new polyhedra at every 45000 time steps, representing their descent into a void environment. The primary objective is to illustrate the process of generating a lattice of polyhedra within a confined area. Additionally, we demonstrate the application of a series of operators to initialize various fields associated with the newly generated polyhedra. This example serves as a practical guide for setting up and executing simulations involving dynamic polyhedra generation and manipulation within defined spatial boundaries.

.. |ex1pstart| image:: ../_static/generator_start.png
   :width: 250pt

.. |ex1pend| image:: ../_static/generator_end.png
   :width: 250pt

+--------------------------+--------------------------+
| .. centered:: Polyhedra Generation Frequency        |
+--------------------------+--------------------------+
| .. centered:: Start      | .. centered:: End        |
+==========================+==========================+
| |ex1pstart|              | |ex1pend|                |
+--------------------------+--------------------------+

Example 2: Octahedra in Rotating Drum
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

In this DEM simulation, we observe the dynamics of 125 octahedra as they descend into a rotating drum. The interaction between these discrete particles, governed by defined the Hooke law. The second test case contains 25,000 octahedra (yellow) and 25,000 hexapods (blue).

.. |ex2pstart| image:: ../_static/octahedra_rotating_drum_start.png
   :width: 250pt

.. |ex2pend| image:: ../_static/octahedra_rotating_drum_end.png
   :width: 250pt

.. |ex2pmixtestart| image:: ../_static/ExaDEM/rotating_drum_mixte_start.png
   :width: 250pt

.. |ex2pmixteend| image:: ../_static/ExaDEM/rotating_drum_mixte_end.png
   :width: 250pt

+--------------------------+--------------------------+
| .. centered:: Polyhedra Generation Frequency        |
+--------------------------+--------------------------+
| .. centered:: Start      | .. centered:: End        |
+==========================+==========================+
| |ex2pstart|              | |ex2pend|                |
+--------------------------+--------------------------+
| |ex2pmixtestart|         | |ex2pmixteend|           |
+--------------------------+--------------------------+


Example 3: Hexapods in Ball 
^^^^^^^^^^^^^^^^^^^^^^^^^^^

This DEM simulation example illustrates the gravitational descent of 64 hexapods within a large sphere. The primary environment consists of a spherical enclosure with a radius of 20 units and centered at (2, 2, 0). As the hexapods descend under gravity within this enclosure, they encounter two additional spherical obstacles. The first obstacle (ballà, represented as a small yellow ball with a radius of 3 units and centererd at (2,2,-5). The second ball centered at (2,2,-20) with a radius of 7 units, depicted as a large orange ball, intersects the surface of the primary blue sphere, adding complexity to the obstacle configuration. Through this simulation, exaDEM code shows its capability to manage particle interactions with various obstacles (balls). Additionally, it showcases the versatility of drivers within the code, which can be employed to define both simulation boundary conditions and obstacles.

.. |ex3pstart| image:: ../_static/ExaDEM/polyhedra_ball_start.png
   :width: 250pt

.. |ex3pend| image:: ../_static/ExaDEM/polyhedra_ball_end.png
   :width: 250pt

+--------------------------+--------------------------+
| .. centered:: Hexapods in Ball                      |
+--------------------------+--------------------------+
| .. centered:: Start      | .. centered:: End        |
+==========================+==========================+
| |ex3pstart|              | |ex3pend|                |
+--------------------------+--------------------------+


Example 4: Polyhedra Wth STL Mesh (Box)
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

This simulation example illustrates the use of stl file with polyhedra. In this simulation, we drop a set of polyhedra (hexapods, octahedra, or both) by gravity into an open box to fill it completely. 

.. |ex4pstarthexa| image:: ../_static/ExaDEM/stl_hexa_start.png
   :width: 250pt

.. |ex4pendhexa| image:: ../_static/ExaDEM/stl_hexa_end.png
   :width: 250pt

.. |ex4pstartocta| image:: ../_static/ExaDEM/stl_octa_start.png
   :width: 250pt

.. |ex4pendocta| image:: ../_static/ExaDEM/stl_octa_end.png
   :width: 250pt

.. |ex4pstartmixte| image:: ../_static/ExaDEM/stl_mixte_start.png
   :width: 250pt

.. |ex4pendmixte| image:: ../_static/ExaDEM/stl_mixte_end.png
   :width: 250pt


+--------------------------+--------------------------+
| .. centered:: Polyhedra With STL Mesh               |
+--------------------------+--------------------------+
| .. centered:: Start      | .. centered:: End        |
+==========================+==========================+
| |ex4pstarthexa|          | |ex4pendhexa|            |
+--------------------------+--------------------------+
| |ex4pstartocta|          | |ex4pendocta|            |
+--------------------------+--------------------------+
| |ex4pstartmixte|         | |ex4pendmixte|           |
+--------------------------+--------------------------+


Example 5 : Funnel
^^^^^^^^^^^^^^^^^^

This simulation example illustrates the gravitational drop of a set of 1.3 million hexapods into a funnel. The funnel is represented using a mesh of faces (STL mesh). Interactions between spheres, as well as between spheres and the funnel, are modeled using Hooke's law.

.. |ex5pstarthalf| image:: ../_static/ExaDEM/funnel_half_start.png
   :width: 250pt

.. |ex5pendhalf| image:: ../_static/ExaDEM/funnel_half_end.png
   :width: 250pt


.. |ex5pstartfull| image:: ../_static/ExaDEM/funnel_full_start.png
   :width: 250pt

.. |ex5pendfull| image:: ../_static/ExaDEM/funnel_full_end.png
   :width: 250pt

+--------------------------+--------------------------+
| .. centered:: Polyhedra With a Funnel               |
+--------------------------+--------------------------+
| .. centered:: Start      | .. centered:: End        |
+==========================+==========================+
| |ex5pstarthalf|          | |ex5pendhalf|            |
+--------------------------+--------------------------+
| |ex5pstartfull|          | |ex5pendfull|            |
+--------------------------+--------------------------+
