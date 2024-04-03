List of DEM operators
=====================

In this section, we describe the main ExaDEM operators and the operators used from ExaNBody. Each operator description is accompanied by a general description, a slot description and an example of use in yaml that you can include in your simulations.

Global Operators
----------------

Some operators from the default `exaNBody` operator list. Here's a list of the main operators that will be useful for your simulations.

* Operator `domain` : see @exaNBody in plugin @exanbIOPlugin
    * `cell_size` : The cell size will be approximately the value given by the possible subdivision of the grid covering the simulation volume. Cell size must be greater than twice the radius of the largest sphere. Note that cell size can greatly influence performance. 
    * `periodic` : Define whether or not the boundary conditions of the system under study are periodic along the (Ox), (Oy) and (Oz) axes.

YAML example:

.. code-block:: yaml
    
   domain:
      cell_size: 2 m
      periodic: [false,true,false]

* Operator `global` :  see @exaNBody in plugin @exanbCorePlugin
    * `simulation_dump_frequency` : Writes an mpiio file each time the number of iterations modulo this number is true.
    * `simulation_paraview_frequency` : Writes an paraview file each time the number of iterations modulo this number is true.
    * `simulation_log_frequency` : Prints logs each time the number of iterations modulo this number is true.
    * `simulation_end_iteration` : Total number of iterations
    * `dt` : Is the integration time value
    * `rcut_inc` : Corresponds to the Verlet radius used by the Verlet list method. Note that the smaller the radius, the more often verlet lists are reconstructed. On the other hand, the larger the radius, the more expensive it is to build the verlet lists.
    * `friction_rcut` : This radius is used to construct lists containing the value of friction between sphere pairs. This radius must be `>= rcut + rcut_inc`.

YAML example:

.. code-block:: yaml
    
   global:
      simulation_dump_frequency: -1
      simulation_end_iteration: 100000
      simulation_log_frequency: 1000
      simulation_paraview_frequency: 5000
      dt: 0.00005 s
      rcut_inc: 0.01 m
      friction_rcut: 1.1 m



Force Field Operators
---------------------




Driver Operators For Spheres
----------------------------

Rigid Wall Operator
^^^^^^^^^^^^^^^^^^^



Field Mutator Operators
-----------------------


This repertory plugin only provides operators for modifying fields, especially at initialization. The following operators are based on the functor `set` and initialize one ore more fields: 

* set_densities_multiple_materials: 
   * [std::vector<double>] `densities`
   * Comment: mass is deduced from the density and radius
* set_density
   * [double] `density`
* set_homothety
   * [double] `homothety`
* set_material_properties
   * [uint8_t] `type`
   * [double] `rad`
   * [double] `density`
   * [Quaternion] `quat`
* set_quaternion
   * [Quaternion] `quat` 
* set_radius
   * [double] `rad`
* set_radius_multiple_materials
   * [std::vector<double>]` radius` (list of radius according to types)
* set_rand_vrot_arot:
   * [double] `var_vrot` (variance), default = 0
   * [double] `var_arot` (variance), default = 0
   * [Vec3d] `mean_arot` (mean), default = {0,0,0}
   * [Vec3d] `mean_vrot` (mean), default = {0,0,0}
   * Comment : This operator set the angular acceleration and velocity using a normal distribution
* set_rand_velocity
   * [double] `var` (variance), default = 0
   * [Vec3d] `mean`, default = {0,0,0}

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

See a minimal example to add your own mutator_field operator in the tutorial section.


