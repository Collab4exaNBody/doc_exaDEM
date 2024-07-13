Fields
======

In this section, we provide an overview of ``ExaDEM`` operators associated with ``field`` mutation. This section is divided into two subsections: ``field`` operators applicable to particles of any type, and ``field`` operators specifically designed for polyhedra.


Field Operators For All Particles
---------------------------------


This repertory plugin only provides operators for modifying fields, especially at initialization. The following operators are based on the functor `set` and initialize one ore more fields: 

* ``set_densities_multiple_materials``: 
   * [std::vector<double>] `densities`
   * Comment: mass is deduced from the density and radius
* ``set_density``:
   * [double] `density`
* ``set_homothety``:
   * [double] `homothety`
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
   * [std::vector<double>]` radius` (list of radius according to types)
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
* ``inertia_from_shape`` : This operator deduces the particle intertia from the shape constant I/M and the particle mass.
* ``radius_from_shape`` : This operator computes the maximum radius cutoff in function of shape types and stores the radius cutoff for every particles corresponding to their shape types.

