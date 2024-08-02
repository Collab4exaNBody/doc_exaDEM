Polyhedron
==========

In this section, we will describe the various information used to build simulations with polyhedron particles.

Overview
^^^^^^^^

The polyhedra implemented in ``ExaDEM`` are sphero-polyhedra, i.e. the vertices of the polyhedra are considered as spheres. To achieve this, ``ExaDEM`` incorporates many of the features of the ``Rockable`` DEM code developed at CNRS (https://github.com/richefeu/rockable, https://richefeu.github.io/rockable/quickStart.html). ``ExaDEM`` relies in particular on a ``Shape`` class containing information about the polyhedron (vertex, edge, face and minskowski radius) and an interaction class used to qualify a contact between polyhedra.


Shape
^^^^^

The ``Shape`` class provides all the information on vertices, edges and faces, but it also provides other support to speed up calculations, such as ``OBB`` sets for each type of information. ``ExaDEM`` provides many features linked to the ``Shape`` class, such as reading ``.shp`` files (the format used by ``Rockable``), as well as other functions such as outputting a ``.vtk`` file of the shape in question.

This class is defined by its:

* m_vertices: List of vertices
* m_edges: List of edges
* m_faces: List of fases
* m_radius: Minskowki radius
* m_volume: Volume
* m_inertia_on_mass: Intertia coeff
* m_name: name, default is undefined
* obb: Oriented Bounded Box of the polyhedron
* m_obb_vertices: List of ``OBB`` for each vertex (only for ``STL Mesh``)
* m_obb_edges: List of ``OBB`` for each edge
* m_obb_faces: List of ``OBB`` for each face

.. note::
	OBB (Oriented Bounded Boxes) are enlarged of the Minskowki radius.

.. note::
	By default, every shapes are stored in a list of shapes, and the maximum cut-off radius are deduced from these shapes. Note that a cut-off radius that is too large can drastically reduce simulation performance. That's why, do not put big shapes using the classical way (i.e. ``read_shape_file``), big shapes should be defined as ``drivers``.

Shape example (octahedron, 6 vertices, 12 edges and 8 faces): 
	
.. code-block:: bash

  <
  name Octahedron
  radius 0.1
  preCompDone y
  nv 6
  0.2310789034541148 -0.2310789034541148 0.0
  0.2310789034541148 0.2310789034541148 0.0
  0.0 0.0 0.32679491924311227
  -0.2310789034541148 -0.2310789034541148 0.0
  -0.2310789034541148 0.2310789034541148 0.0
  0.0 0.0 -0.32679491924311227
  ne 12
  0 1
  2 1
  2 0
  0 3
  2 3
  3 4
  4 2
  4 1
  5 0
  5 1
  5 4
  5 3
  nf 8
  3 0 1 2 
  3 2 3 4 
  3 1 2 4 
  3 0 2 3 
  3 0 5 1 
  3 0 5 3 
  3 3 5 4 
  3 4 5 1 
  obb.extent 0.33107890345411484 0.33107890345411484 0.4267949192431123
  obb.e1 1.0 0.0 0.0
  obb.e2 0.0 1.0 0.0
  obb.e3 0.0 0.0 1.0
  obb.center 0.0 0.0 0.0
  position 0.0 0.0 0.0
  orientation 1.0 0.0 0.0 0.0
  volume 0.16666666666666666
  I/m 0.04999999999999999 0.04999999999999999 0.04999999999999999
  >

Or a sphere (1 vertex, 0 edge, 0 face):

.. code-block:: bash

  <
  name alpha1
  radius 0.5
  preCompDone y
  nv 1
  0 0 0
  ne 0
  nf 0
  obb.extent 0.5 0.5 0.5
  obb.e1 1 0 0
  obb.e2 0 1 0
  obb.e3 0 0 1
  obb.center 0 0 0
  volume 0.523598775598299
  I/m 0.1 0.1 0.1
  >

It's important to note that using a shape of a spherical particle with a polyhedron configuration instead of directly using a sphere configuration decreases overall performance due to unnecessary calculations such as applying an orientation to a vertex. We have observed that in this case, simulations are about 2 to 3 times slower. 

* Operator Name: ``read_shape_file``
* Description: This operator initialize shapes data structure from a shape input file.
* Parameter:

  * filename: Input file name (.stl)


Interaction / Contact
^^^^^^^^^^^^^^^^^^^^^

The ``exaDEM::Interaction`` class in ``ExaDEM`` is used to model various types of interactions between polyhedra and between polyhedra and ``drivers``. This class serves as a crucial component for identifying two elements within the data grid and characterizing the type of interaction between them.

**Interaction Class Attributes:**

* id_i and id_j: Id of both polyhedra.
* cell_i and cell_j: Indices of the cells containing the interacting polyhedra.
* p_i and p_j: Positions of the polyhedra within their respective cells.
* sub_i and sub_j: Indices of the vertex, edge, or face of the polyhedron involved in the interaction.
* type: Type of interaction (integer). See Interaction Glossary.
* friction and moment: Storage used for temporary computations.


.. note::
  When the interaction involves a polyhedron and a ``driver``, particle j is used to locate the ``driver``. In this scenario, cell_j represents the index of the ``driver``. If the ``driver`` utilizes a shape, such as with ``STL meshes``, sub_j is also utilized to store the index of the vertex, edge, or face.


.. list-table:: Glossary of ``Interaction`` types
   :widths: 10 25 65
   :header-rows: 1

   * - Value
     - Type 
     - Description
   * - 0
     - Vertex - Vertex
     - Contact between two vertices of two different polyhedra
   * - 1
     - Vertex - Edge
     - Contact between a vertex and an edge of two different polyhedra
   * - 2
     - Vertex - Face
     - Contact between a vertex and a face of two different polyhedra
   * - 3
     - Edge - Edge
     - Contact between two edges of two different polyhedra
   * - 4
     - Vertex - Cylinder
     - Contact between a vertex of a polyhedron and a cylinder
   * - 5
     - Vertex - Surface
     - Contact between a vertex of a polyhedron and a rigid surface or wall
   * - 6
     - Vertex - Ball
     - Contact between a vertex of a polyhedron and a ball / sphere
   * - 7
     - Vertex - Vertex (STL)
     - Contact between a vertex of a polyhedron and a vertex of a stl mesh
   * - 8
     - Vertex - Edge (STL)
     - Contact between a vertex of a polyhedron and an edge of a stl mesh
   * - 9
     - Vertex - Face (STL)
     - Contact between a vertex of a polyhedron and a face of a stl mesh
   * - 10
     - Edge - Edge (STL)
     - Contact between a edge of a polyhedron and a edge of a stl mesh
   * - 11
     - Vertex (STL) - Edge
     - Contact between a vertex of a stl mesh and a edge of a polyhedron
   * - 12
     - Vertex (STL) - Face
     - Contact between a vertex of a stl mesh and a face of a polyhedron

**Interaction Class Usage:**

To retrieve data associated with a specific interaction between two polyhedra, the attributes of the ``exaDEM::Interaction`` class are used to identify cells, positions, and interaction types. Theses informations are then utilized within simulation computations to accurately model interactions between polyhedra, considering the interaction type.

These interactions are utilized as a level of granularity for intra-node parallelization, applicable to both ``CPU`` and upcoming ``GPU`` implementations. The interactions are populated within the ``nbh_polyhedron`` operator and subsequently processed in the ``hooke_polyhedron`` operator.


In summary, the ``exaDEM::Interaction`` class provides a crucial data structure for managing interactions between polyhedra and drivers within DEM simulations. By storing information such as cell numbers, positions, and interaction types, it enables precise modeling of physical interactions between simulated objects.

**Grid Of Interactions:**

In ``ExaDEM``, interactions are stored in the form of a grid of cells (AOSOA), the cell (SOA) then containing a ``GridExtraDynamicDataStorageT``, i.e. a data structure similar to a vector of ``Interactions`` + particle information vector. This data structure facilitates the migration of information between ``MPI`` processes when the interaction is considered to be always active (i.e. the two polyhedra are always in contact from one time step to the next). For more details in code, see `src/polyhedra/include/exaDEM/interaction/grid_cell_interaction.hpp` and ``extra_storage`` package in ``ExaNBody``.

**Classifier:**

To improve the implementation of kernels linked to ``GPU`` interactions, ``exaDEM`` relies on the `classifier` class, which sorts all interactions by type in an ``SOA``, so that several kernels can be launched, each dealing with the same type of interaction. The aim is to limit instruction divergence between ``GPU`` threads.

It's important to point out that this data structure complements the interaction grid. The main idea is to classify and unclassify interaction information as long as the data has not changed (``cell migration``, ``move particle``, ``IO``). To achieve this, we use two operators: ``classify`` and ``unclassify``.

Using the classifier is currently the default strategy in exaDEM for spheres and polyhedra.
