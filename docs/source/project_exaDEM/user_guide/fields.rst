Particle Fields
===============

This page covers ``ExaDEM``'s ``field``-mutation operators, in three parts: the unified ``set_fields`` operator (for both spheres and polyhedra), operators that work on particles of any type, and operators specific to polyhedra.


Unified Field Operators
-----------------------

.. warning::

  ``set_fields`` accepts many parameter combinations, and not all of them have been exhaustively
  tested. If a combination produces unexpected results, please report it.

* Operator name: ``set_fields``:
* Parameters ``REQUIRED``:
   * [bool] `polyhedra`: Whether the particles are polyhedra (``true``) or spheres (``false``).
   * [std::vector<string>] `type`: Particle type names.
* Parameters ``OPTIONAL``:
   * [std::vector<double>] `density`: List of density values. If not defined, density is 1.
   * [std::vector<double>] `radius`: List of radius values. If not defined, radius is 0.5 for spheres, do not define it for polyhedra.
   * [std::vector<Vec3d>] `velocity`: List of velocity values. If not defined, velocity is [0,0,0].
   * [std::vector<double>] `sigma_velocity`: Standard deviation (sigma). If not defined, the normal distribution is not applied.
   * [std::vector<Vec3d>] `angular_velocity`: List of angular velocity values. If not defined, angular velocity is [0,0,0].
   * [std::vector<double>] `sigma_angular_velocity`: Standard deviation (sigma). If not defined, the normal distribution is not applied.
   * [std::vector<Quaternion>] `quaternion`: List of orientations. If not defined, quaternion is [w = 1,0,0,0].
   * [std::vector<bool>] `random_quaternion`: Whether the orientation is randomized. If not defined, random is false.
   * [std::vector<uint32_t>] `group`: Group index associated to each particle type (same order as `type`). If not defined, group is 0 for all particles.

.. note::

  For spheres, you need to call the ``particle_type`` operator to create the particle map required by ``set_fields``.

.. note::

  Since ``exaDEM-1.2.3``, the ``group`` field is used by the multi-material contact operators (``multimat_contact_params``, ``drivers_contact_params``, ``inner_bond_params``) to select which set of contact parameters applies between two particles. Before ``1.2.3``, the particle ``type`` itself was directly used to look up these contact parameters. With ``group``, several particle types can share the same group and therefore reuse the same contact parameters, decoupling the (shape-related) ``type`` from the (contact-law-related) ``group``. See :ref:`force_field_multi_material` on the Force Field page for details. Groups can also be (re)assigned after initialization with the ``set_group`` operator, below.

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
     group:          [       0,       1 ]
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
     type:              [ alpha3, Octahedron ]
     group:             [      0,          1 ]
     velocity:          [ [0,0,0],   [0,0,0] ]
     sigma_velocity:    [     0.1,       0.1 ]
     random_quaternion: [    true,      true ]

Field Operators For All Particles
---------------------------------


This plugin provides operators for modifying fields, mainly at initialization. The following operators are based on the ``set`` functor and initialize one or more fields:

* ``set_densities_multiple_materials``: Applies a different density to each particle type. Mass is deduced from density and radius.
   * [std::vector<double>] `densities`
* ``set_density``: Applies the same density to every particle. Use ``set_densities_multiple_materials`` above instead if densities should vary by particle type.
   * [double] `density`
* ``set_homothety``: Sets the same homothety (scale factor) on every particle in a region.
   * [double] `homothety`
* ``set_type``: Sets the same particle type on every particle in a region.
   * [uint32_t] `type`
* ``set_material_properties``: Sets radius, density, and orientation together, in one call, for every particle in a region.
   * [uint32_t] `type`
   * [double] `rad`
   * [double] `density`
   * [Quaternion] `quat`
* ``set_quaternion``: Sets the same orientation on every particle in a region, or a random orientation for each.
   * [Quaternion] `quat`
   * [bool] `random`, default is false.
* ``set_radius``: Sets the same radius on every particle in a region.
   * [double] `rad`
* ``set_radius_multiple_materials``: Applies a different radius to each particle type.
   * [std::vector<double>] `radius` (list of radii according to types)
* ``set_rand_vrot_arot``: Draws angular velocity and angular acceleration from a normal distribution, for every particle.
   * [double] `var_vrot` (variance), default = 0
   * [double] `var_arot` (variance), default = 0
   * [Vec3d] `mean_arot` (mean), default = {0,0,0}
   * [Vec3d] `mean_vrot` (mean), default = {0,0,0}
* ``set_rand_velocity``: Draws velocity from a normal distribution, for every particle in a region.
   * [double] `var` (variance), default = 0
   * [Vec3d] `mean`, default = {0,0,0}
* ``update_inertia``: Recomputes the inertia field from mass and radius (:math:`0.4 \cdot mass \cdot radius^2`), for every particle in a region.
* ``set_group`` (since ``exaDEM-1.2.3``):
   * [std::vector<string>] `type` ``REQUIRED``: List of particle type names.
   * [std::vector<uint32_t>] `group` ``REQUIRED``: Group index associated to each type (same order as `type`).
   * [uint32_t] `n_groups` ``OUTPUT``: Number of distinct groups (max group index + 1), written back for downstream operators such as ``multimat_contact_params``, ``drivers_contact_params`` and ``check_group_completeness``.
   * Comment: Assigns the ``group`` field to every particle according to its type. It can be used to (re)define groups after ``set_fields`` has been called, without having to redefine every other field.

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

``set_group`` YAML example:

.. code-block:: yaml

   init_polyhedra:
     - set_group:
        type:  [ alpha3, Octahedron, Cube ]
        group: [      0,          1,    0 ]

   init_spheres:
     - set_group:
        type:  [ Sphere1, Sphere2, Sphere3 ]
        group: [       0,       1,       0 ]

See :ref:`add_mutator_field_operator` in the Tutorial page for a minimal example of how to add your own ``mutator_field`` operator.


Field Operators For Polyhedra
-----------------------------

In this section, we briefly describe ``field`` mutator operators that relate to data contained within the ``shape`` data structure (see :ref:`polyhedra_shape` on the R-Shape / Polyhedron page for more details).


* ``density_from_shape`` : This operator deduces the particle mass from the shape volume and the particle density.
   * [double] `density`
* ``inertia_from_shape`` : This operator deduces the particle inertia from the shape constant I/M and the particle mass.
* ``radius_from_shape`` : This operator computes the maximum radius cutoff as a function of shape type, and stores it on every particle according to its shape type.

.. note::

  It is possible to specify the ``region`` slot to apply the following kernel to a special spatial area: ``radius_from_shape``.



Rigid Frame Operator
--------------------

The ``compute_frame_velocity`` operator drives a group of particles (a *frame*, e.g. the loading plate of a mechanical test) as a single rigid body. For each component of the frame velocity, the user chooses between:

* ``velocity`` control: the velocity is imposed;
* ``force`` control: the velocity is obtained from the motion of a virtual rigid body of mass :math:`M`, driven by a target force:

.. math::

  M \frac{dv}{dt} = F_{target}(t) + F_{specimen} - c \, v

where :math:`F_{specimen}` is the sum of the forces acting on the particles of the frame region (i.e. the force exerted by the sample on the frame), :math:`F_{target}` is the force applied by the "machine" on the frame and :math:`c` is a viscous damping coefficient. The velocity is integrated with an explicit Euler scheme using the simulation time step.

The operator does **not** modify the particles: it only computes the ``frame_velocity`` slot. The velocity is then applied with the existing operators ``freeze_particles`` and ``set_velocity`` (with its ``value`` slot rebound to ``frame_velocity``).

* Operator name: ``compute_frame_velocity``
* Parameters ``REQUIRED``:
   * [ParticleRegionCSG] `region`: Frame region, e.g. ``TOP`` or ``TOP or LEFT_UP``.
   * [double] `frame_mass`: Virtual mass :math:`M` of the frame (used by force-controlled components).
* Parameters ``OPTIONAL``:
   * [std::vector<string>] `control_mode`: Control mode of each component (x, y, z), ``velocity`` or ``force``. Default is ``[velocity, velocity, velocity]``.
   * [Vec3d] `imposed_velocity`: Velocity of the velocity-controlled components. Default is [0,0,0].
   * [Vec3d] `target_force`: Target force applied **by the machine on the frame** (force-controlled components). Default is [0,0,0].
   * [long] `force_ramp_start`: Timestep at which the linear force ramp starts. Default is 0.
   * [long] `force_ramp_steps`: Number of timesteps of the linear ramp from 0 to `target_force` (0: no ramp, the full force is applied at once). Default is 0.
   * [double] `frame_damping`: Viscous damping coefficient :math:`c`. Default is 0.
   * [double] `frame_max_velocity`: Cap on the absolute value of each force-controlled velocity component (<= 0: no cap). Default is -1.
   * [long] `frame_log_frequency`: Prints the frame state (number of particles, target force, specimen force, velocity) every n timesteps (<= 0: never). Default is 1000.
* Parameters ``INPUT_OUTPUT`` / ``OUTPUT``:
   * [Vec3d] `frame_velocity`: Current frame velocity. This is the state of the virtual rigid body; it is shared between all the instances of the operator, which allows switching between several loading stages without discontinuity.
   * [Vec3d] `frame_force`: Force exerted by the sample on the frame (:math:`F_{specimen}`), available for other operators.

.. note::

  * Place ``compute_frame_velocity`` in ``compute_force`` **after** the contact and inner bond operators (so that the particle forces are complete) and **before** ``freeze_particles`` / ``set_velocity``.
  * The `region` of ``compute_frame_velocity`` must contain **all** the regions that are frozen and moved with the frame. A region that is moved but not included in the sum of the forces is not in equilibrium with the frame.
  * The sign convention follows the forces: with a target force of +F along y on the top frame, the machine pulls the frame upward (tension); in this case :math:`F_{specimen}` is negative.
  * After a sudden failure of the sample, a force-controlled frame accelerates freely. `frame_max_velocity` limits this runaway; it is a numerical device that should be reported when it is active.
  * Choosing :math:`M` and :math:`c`: the frame and the sample behave as a mass-spring system of stiffness :math:`K` (sample stiffness). The period :math:`2\pi\sqrt{M/K}` should be large compared to the time step, and :math:`c \approx 2\sqrt{K M}` gives a critically damped response.

YAML example (spheres, compression of a sample by a force-controlled top frame, see ``example/spheres/impose_frame/movable_frame_force.msp``):

.. code-block:: yaml

  particle_regions:
    - FRAME: { bounds: [ [ -5. , -5. , 19.0 ] , [ 15. , 15. , 25. ] ] }

  frame_box:
    - compute_frame_velocity:
        region: FRAME
        control_mode: [ velocity , velocity , force ]   # x, y, z
        target_force: [ 0. , 0. , -72. ]                # pushes the frame toward -z
        imposed_velocity: [ 0. , 0. , 0. ]
        frame_mass: 1.0
        frame_log_frequency: 1000
    - freeze_particles: { region: FRAME }
    - apply_frame_velocity:
        rebind: { value: frame_velocity }
        body:
          - set_velocity: { region: FRAME }

  compute_force:
    - contact_singlemat_sphere_hooke_cohesive:
       ...
    - frame_box

YAML example (polyhedra, tension of a block of bonded polyhedra: bottom frame fixed, top frame driven by a force ramp along y):

.. code-block:: yaml

  particle_regions:
    - BOT: { bounds: [ [ -5. , -5. , -5. ] , [ 15. ,  3. , 10. ] ] }
    - TOP: { bounds: [ [ -5. , 17. , -5. ] , [ 15. , 25. , 10. ] ] }

  frame_traction:
    - compute_frame_velocity:
        region: TOP
        control_mode: [ velocity , force , velocity ]
        target_force: [ 0. , 150. , 0. ]
        force_ramp_start: 0
        force_ramp_steps: 100000
        frame_mass: 60.
        frame_damping: 3500.
        frame_max_velocity: 0.02
        frame_log_frequency: 200
    - freeze_particles: { region: TOP }
    - apply_frame_velocity:
        rebind: { value: frame_velocity }
        body:
          - set_velocity: { region: TOP }
    - freeze_particles: { region: BOT }

  compute_force:
    - contact_multimat_polyhedron
    - inner_bond_polyhedron
    - frame_traction

YAML example (two loading stages: a lateral force is first ramped up, then kept constant while an axial velocity is imposed; both instances share the same ``frame_velocity`` state, and ``target_force`` / ``force_ramp_steps`` must be identical in both stages):

.. code-block:: yaml

  eval_stage:
    rebind: { result: stage1 }
    body:
      - before_timestep: { end_at: 72000 }

  frame_stage1:
    condition: stage1
    body:
      - compute_frame_velocity:
          region: TOP or LEFT_UP
          control_mode: [ force , force , velocity ]
          target_force: [ 585. , 0. , 0. ]
          force_ramp_steps: 52000
          frame_mass: 60.
          frame_damping: 7000.
          frame_max_velocity: 0.02

  frame_stage2:
    condition: not stage1
    body:
      - compute_frame_velocity:
          region: TOP or LEFT_UP
          control_mode: [ force , velocity , velocity ]
          target_force: [ 585. , 0. , 0. ]
          imposed_velocity: [ 0. , 0.002 , 0. ]
          force_ramp_steps: 52000
          frame_mass: 60.
          frame_damping: 7000.
          frame_max_velocity: 0.02

  frame_loading:
    - eval_stage
    - frame_stage1
    - frame_stage2
    - freeze_particles: { region: TOP or LEFT_UP }
    - apply_frame_velocity:
        rebind: { value: frame_velocity }
        body:
          - set_velocity: { region: TOP or LEFT_UP }

