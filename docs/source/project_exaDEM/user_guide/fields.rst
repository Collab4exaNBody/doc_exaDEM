Particle Fields
===============

In this section, we provide an overview of ``ExaDEM`` operators associated with ``field`` mutation. This section is divided into three subsections: the unified ``field`` operator for spheres and polyhedra, ``field`` operators applicable to particles of any type, and ``field`` operators specifically designed for polyhedra.


Unified Field Operators
-----------------------

.. warning:: 

  This operator has many possible combinations, some of which may be wrong. Please report any doubts.

* Operator name: ``set_fields``:
* Parameters ``REQUIRED``:
   * [bool] `polyhedra`: Define if the kind of particles is polyhedron or sphere.
   * [std::vector<string>] `type`: Particle type names.
* Paramters ``OPTIONAL``:
   * [std::vector<double>] `density`: List of density values. If not defined, density is 1.
   * [std::vector<double>] `radius`: List of radius values. If not defined, radius is 0.5 for spheres, do not define it for polyhedra.
   * [std::vector<Vec3d>] `velocity`: List of velocity values. If not defined, velocity is [0,0,0].
   * [std::vector<double>] `sigma_velocity`: Standard deviation (sigma). If not defined, the normal distribution is not applied.
   * [std::vector<Vec3d>] `angular_velocity`: List of angular velocity values. If not defined, angular velocity is [0,0,0].
   * [std::vector<double>] `sigma_angular_velocity`: Standard deviation (sigma). If not defined, the normal distribution is not applied.
   * [std::vector<Quaternion>] `quaternion`: List of orientations. If not defined, quaternion is [w = 1,0,0,0].
   * [std::vector<bool>] `random_quaternion`: Choice if the orientation is random or not. If not defined, random is false.

.. note::

  For spheres, you need to call particle_type operator to create the particle map required by ``set_fields``.

YAML example (Spheres):

.. code-block:: yaml

  - particle_type:
     type: [  Sphere1,  Sphere2 ]
  - lattice:
     structure: BCC
     types: [ Sphere1,  Sphere2 ]
     size: [ 1.0 , 1.0 , 1.0 ]
  - set_fields:
     polyhedra: false
     type:           [ Sphere1, Sphere2 ]
     radius:         [     0.5,    0.25 ]
     density:        [    0.02,    0.01 ]
     velocity:       [ [0,0,0], [0,0,0] ]
     sigma_velocity: [    0.01,    0.01 ]

YAML example (Polyhedra):

.. code-block:: yaml

  - read_shape_file:
     filename: alpha3.shp
  - read_shape_file:
     filename: octahedron.shp
  - lattice:
      structure: BCC
      types: [alpha3,Octahedron ]
      size: [ 1.5 , 1.5 , 1.5 ]
      repeats: [ 30 , 30 , 30 ]
      enlarge_bounds: 0.0 m
  - set_fields:
     polyhedra: true
     type:              [ alpha3, Octahedron]
     velocity:          [[0,0,0],    [0,0,0]]
     sigma_velocity:    [    0.1,        0.1]
     random_quaternion: [   true,       true]

Field Operators For All Particles
---------------------------------


This repertory plugin only provides operators for modifying fields, especially at initialization. The following operators are based on the functor `set` and initialize one or more fields: 

* ``set_densities_multiple_materials``: 
   * [std::vector<double>] `densities`
   * Comment: mass is deduced from the density and radius
* ``set_density``:
   * [double] `density`
* ``set_homothety``:
   * [double] `homothety`
* ``set_type``:
   * [uint32_t] `type`
* ``set_material_properties``:
   * [uint8_t] `type`
   * [double] `rad`
   * [double] `density`
   * [Quaternion] `quat`
* ``set_quaternion``:
   * [Quaternion] `quat`
   * [bool] `random`, default is false. 
* ``set_radius``:
   * [double] `rad`
* ``set_radius_multiple_materials``:
   * [std::vector<double>]` radius` (list of radii according to types)
* ``set_rand_vrot_arot``:
   * [double] `var_vrot` (variance), default = 0
   * [double] `var_arot` (variance), default = 0
   * [Vec3d] `mean_arot` (mean), default = {0,0,0}
   * [Vec3d] `mean_vrot` (mean), default = {0,0,0}
   * Comment : This operator set the angular acceleration and velocity using a normal distribution
* ``set_rand_velocity``:
   * [double] `var` (variance), default = 0
   * [Vec3d] `mean`, default = {0,0,0}
* ``update_inertia``

.. note::

  It is possible to specify the ``region`` slot to apply the following kernel to a special spatial area: ``set_homothety``, ``set_type``, ``set_quaternion``, ``set_rand_velocity``, ``set_material_properties``, ``set_radius``, and ``update_inertia``.

YAML example:


.. code-block:: yaml

   - set_radius:
      rad: 0.5
   - set_quaternion
   - set_rand_velocity:
      var: 0.1
      mean: [0.0,0.0,0.0]
   - set_density:
      density: 0.02
   - set_rand_vrot_arot

Explore a minimal example provided in the "tutorial" section to understand how you can add your own mutator_field operator.

Field Operators For Polyhedra
-----------------------------

In this section, we briefly describe ``field`` mutator operators that relate to data contained within the ``shape`` data structure (see the Polyhedra Section for more details).


* ``density_from_shape`` : This operator deduces the particle mass from the shape volume and the particle density.
   * [double] `density`
* ``inertia_from_shape`` : This operator deduces the particle inertia from the shape constant I/M and the particle mass.
* ``radius_from_shape`` : This operator computes the maximum radius cutoff in function of shape types and stores the radius cutoff for every particle corresponding to their shape types.

.. note::

  It is possible to specify the ``region`` slot to apply the following kernel to a special spatial area: ``radius_from_shape``.

