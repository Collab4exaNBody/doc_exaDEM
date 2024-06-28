Spheres
=======

In this section, we will describe the various information used to build simulations with spheres particles. Note that ``exaDEM::Interaction`` and ``Shape`` are describe in the section : Polyhedron.

Interaction / Contact
^^^^^^^^^^^^^^^^^^^^^

The ``exaDEM::Interaction`` class in ``ExaDEM`` is used to model various types of interactions between sphres and between spheres and ``drivers``. This class serves as a crucial component for identifying two elements within the data grid and characterizing the type of interaction between them. Note that, for the sake of consistency between the sphere and sphere sections, we use the same interaction structure, and that the type field that qualifies the nature of the interaction has the same values as for polyhedra. 

**Interaction Class Attributes:**

* id_i and id_j: Id of both spheres.
* cell_i and cell_j: Indices of the cells containing the interacting spheres.
* p_i and p_j: Positions of the spheres within their respective cells.
* sub_i and sub_j: Indices of the vertex, edge, or face of the sphere involved in the interaction.
* type: Type of interaction (integer). See Interaction Glossary.
* friction and moment: Storage used for temporary computations.


.. note::
  When the interaction involves a sphere and a ``driver``, particle j is used to locate the ``driver``. In this scenario, cell_j represents the index of the ``driver``. If the ``driver`` utilizes a shape, such as with ``STL meshes``, sub_j is also utilized to store the index of the vertex, edge, or face.


.. list-table:: Glossary of ``Interaction`` types for Spheres
   :widths: 10 25 65
   :header-rows: 1

   * - Value
     - Type 
     - Description
   * - 0:
     - Sphere - Sphere
     - Contact between two vertices of two different spheres
   * - 4
     - Sphere - Cylinder
     - Contact between a vertex of a sphere and a cylinder
   * - 5
     - Sphere - Surface
     - Contact between a vertex of a sphere and a rigid surface or wall
   * - 6
     - Sphere - Ball
     - Contact between a vertex of a sphere and a ball / sphere
   * - 7
     - Sphere - Vertex (STL)
     - Contact between a vertex of a sphere and a vertex of a stl mesh
   * - 8
     - Sphere - Edge (STL)
     - Contact between a vertex of a sphere and an edge of a stl mesh
   * - 9
     - Sphere - Face (STL)
     - Contact between a vertex of a sphere and a face of a stl mesh

**Interaction Class Usage With Spheres:**

To retrieve data associated with a specific interaction between two spheres, the attributes of the ``exaDEM::Interaction`` class are used to identify cells, positions, and interaction types. Theses informations are then utilized within simulation computations to accurately model interactions between spheres, considering the interaction type.

These interactions are utilized as a level of granularity for intra-node parallelization, applicable to both ``CPU`` and upcoming ``GPU`` implementations. The interactions are populated within the ``nbh_sphere`` operator and subsequently processed in the ``hooke_sphere`` operator.


In summary, the ``exaDEM::Interaction`` class provides a crucial data structure for managing interactions between spheres and drivers within DEM simulations. By storing information such as cell numbers, positions, and interaction types, it enables precise modeling of physical interactions between simulated objects.

