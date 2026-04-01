Spherical Particle
==================

In this section, we will describe the various information used to build simulations with sphererical particles. Note that ``exaDEM::Interaction`` and ``Shape`` are described in the section: Polyhedron.

.. _interaction_type_sphere:

Spheres - Interaction / Contact
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

The ``exaDEM::Interaction`` class in ``ExaDEM`` is used to model various types of interactions between spheres themselves and between spheres and ``drivers``. This class serves as a crucial component for identifying two elements within the data grid and characterizing the type of interaction between them. Note that, for the sake of consistency between the sphere and sphere sections, we use the same interaction structure, and that the type field that qualifies the nature of the interaction has the same values as for polyhedra. 

**Interaction Class Attributes:**

* :math:`id_i` and :math:`id_j`: Id of both spheres.
* :math:`cell_i` and :math:`cell_j`: Indices of the cells containing the interacting spheres.
* :math:`p_i` and :math:`p_j`: Positions of the spheres within their respective cells.
* :math:`sub_i` and :math:`sub_j`: Indices of the vertex, edge, or face of the sphere involved in the interaction.
* type: Type of interaction (integer). See Interaction Glossary.
* friction and moment: Storage used for temporary computations.


.. note::
  When the interaction involves a sphere and a ``driver``, particle j is used to locate the ``driver``. In this scenario, cell_j represents the index of the ``driver``. If the ``driver`` utilizes a shape, such as with ``RShapeDriver``, sub_j is also utilized to store the index of the vertex, edge, or face.


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
     - Sphere - Vertex (Driver)
     - Contact between a vertex of a sphere and a vertex of a RShape Driver
   * - 8
     - Sphere - Edge (Driver)
     - Contact between a vertex of a sphere and an edge of a RShape Driver
   * - 9
     - Sphere - Face (Driver)
     - Contact between a vertex of a sphere and a face of a RShape Driver

**Interaction Class Usage With Spheres:**

To retrieve data associated with a specific interaction between two spheres, the attributes of the ``exaDEM::Interaction`` class are used to identify cells, positions, and interaction types. These informations are then utilized within simulation computations to accurately model interactions between spheres, considering the interaction type.

These interactions are utilized as a level of granularity for intra-node parallelization, applicable to both ``CPU`` and upcoming ``GPU`` implementations. The interactions are populated within the ``nbh_sphere`` operator and subsequently processed in the ``contact_sphere`` operator.

In summary, the ``exaDEM::Interaction`` class provides a crucial data structure for managing interactions between spheres and drivers within DEM simulations. By storing information such as cell numbers, positions, and interaction types, it enables precise modeling of physical interactions between simulated objects.

Operators
^^^^^^^^^

randomize_radius
~~~~~~~~~~~~~~~~

The ``randomize_radius`` operator randomly modifies the radius of spherical
particles and updates their physical properties accordingly.

For each particle, a new radius is sampled from a Gaussian distribution
centered on the current radius. The width of the distribution is controlled
by the ``relative_deviation`` parameter. After the new radius is computed,
the particle mass and inertia tensor are updated consistently assuming that
the particle density remains constant.

The radius variation is bounded in order to avoid unrealistic values.

**Parameters**

* ``relative_deviation`` (double, required)

  Relative deviation applied to the particle radius.  
  For example, ``0.2`` corresponds to a variation of approximately
  :math:`\pm 20\%` around the current radius.

* ``allow_exceed`` (bool, optional, default = true)

  Controls whether the new radius is allowed to exceed the original radius.

  * ``true``: the radius may increase or decrease within the allowed range.
  * ``false``: the radius cannot become larger than the initial radius.

* ``region`` (optional)

  A ``ParticleRegionCSG`` definition used to restrict the operation to
  particles belonging to a specific region.

* ``particle_regions`` (optional)

  Collection of particle regions used to evaluate the ``region`` expression.

**Example**

Basic usage:

.. code-block:: yaml

  - randomize_radius:
      relative_deviation: 0.2

Allowing the radius to increase beyond its initial value:

.. code-block:: yaml

  - randomize_radius:
      relative_deviation: 0.2
      allow_exceed: true

Applying the operator only to a specific particle region:

.. code-block:: yaml

  - randomize_radius:
      relative_deviation: 0.15
      allow_exceed: true
      region: my_region
