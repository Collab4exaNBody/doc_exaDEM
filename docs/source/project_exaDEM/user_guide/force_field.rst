Force Field
===========

The force field encompasses a broader set of operators and mechanisms responsible for computing forces acting on particles within the particle environment. It includes various types of forces such as gravitational forces, contact forces, and other external influences that affect particle dynamics.


Contact Force Laws
------------------

A **contact law**, in the context of the Discrete Element Method (DEM), is the principle used to calculate the force between two particles from their relative displacement. In DEM simulations, a contact law models the interaction between particles, capturing elastic deformation and linear force behaviors.

There are two kinds of interaction:

- pure contact interaction
- cohesive interaction

The main laws available in exaDEM are:

+--------------+----------------+------------------------------------------------------------------------------------------------------------------------------------+
| Name         | type           | Description                                                                                                                        |
+==============+================+====================================================================================================================================+
| ``hooke``    | pure contact   | Default configuration : elastic linear normal force, normal viscosity force, Coulomb friction tangent force and rolling resistance |
+--------------+----------------+------------------------------------------------------------------------------------------------------------------------------------+
| ``cohesive`` | Cohesive       | Addition of a normal cohesive force depending on the distance between particles                                                    |
+--------------+----------------+------------------------------------------------------------------------------------------------------------------------------------+
| ``dmt``      | Cohesive       | Addition of an adhesive law at contact to model Van der Waals force for hard particles                                             |
+--------------+----------------+------------------------------------------------------------------------------------------------------------------------------------+

The variables required to describe interactions are:

.. list-table::
   :widths: 25 75
   :header-rows: 1

   * - Variable
     - Description
   * - :math:`cp`
     - The contact position
   * - :math:`r_i`
     - The position of particle :math:`i`
   * - :math:`r_j`
     - The position of particle :math:`j`
   * - :math:`v_i`
     - The velocity of particle :math:`i`
   * - :math:`v_j`
     - The velocity of particle :math:`j`
   * - :math:`vrot_i`
     - The angular velocity of particle :math:`i`
   * - :math:`vrot_j`
     - The angular velocity of particle :math:`j`
   * - :math:`m_i`
     - The mass of particle :math:`i`
   * - :math:`m_j`
     - The mass of particle :math:`j`
   * - :math:`\delta_n`
     - The interpenetration / particle overlap

Each kind of interaction also requires additional constants, reflecting for most of them mechanical properties:

.. list-table::
   :widths: 25 75
   :header-rows: 1

   * - Constant
     - Description
   * - :math:`\Delta_t`
     - The timestep increment
   * - :math:`\alpha_n`
     - The damping rate
   * - :math:`k_n`
     - The normal stiffness coefficient
   * - :math:`k_t`
     - The tangential stiffness coefficient
   * - :math:`v_t`
     - The relative tangential velocity
   * - :math:`k_r`
     - The rotational stiffness coefficient
   * - :math:`\mu`
     - The coefficient of friction

``hooke`` law: elastic normal force with tangential friction
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

In the Discrete Element Method (DEM), the equations of motion (translations and rotations) are discretized in time. Only the rigid-body displacements are considered. Small overlaps between the particles are allowed and used as strain variables. The total contact force between particle :math:`i` and particle :math:`j` is given by 

.. math::

 \textbf{f}_{ij} = f_n \textbf{n}  +  \textbf{f}_t

where :math:`f_n` the normal component of the contact force and :math:`\textbf{f}_t` is the tangential force vector. These forces are expressed in the local contact frame :math:`(\textbf{n},\textbf{t},\textbf{s})` as a function of the overlaps and tangential displacements. They are calculated from force laws that generally describe frictional contact interactions. An important feature of DEM is to allow the particles to overlap. This overlap :math:`\delta_n` represents a normal strain localized in the vicinity of the contact point. A simple linear relation is assumed between normal contact force and :math:`\delta_n` . This is consistent with the fact that the overlaps allow for a penalty-based explicit formulation of particle motions, i.e., the elastic repulsion force is mobilized to prevent penalizing the overlap. The condition of particle undeformability implies that the overlaps must stay small compared to particle size. In this linear approximation, the normal component of the contact force is given by 

.. math::

  f_n =  - k_n \delta_n + \nu_n v_n

where :math:`k_n` is the normal stiffness coefficient, :math:`v_n` the normal component of the relative velocity (between particle :math:`i` and particle :math:`j`), and :math:`\nu_n` is the viscous damping coefficient. 
:math:`\nu_n` is related to the restitution coefficient :math:`e_n` by 

.. math::

  \nu_n = \alpha_n \sqrt{2 m_{\text{eff}} v_n}

  \alpha_n = \frac{- \ln{e_n}}{\sqrt{\ln^2{e_n} + \pi^2}}

where:

- :math:`\nu_n` is the viscous damping rate during the collision
- :math:`\alpha_n` is the damping parameter, calculated from the restitution coefficient :math:`e_n`
- :math:`m_{\text{eff}}` is the effective mass of the two colliding particles, defined by :math:`m_{\text{eff}} = \frac{m_i m_j}{m_i + m_j}`


.. note::
 
  The formulas are identical to those used in Rockable (see `Rockable Force Laws <https://richefeu.github.io/rockable/forceLaws.html#default-model-keywork-default>`_) but are implemented differently to align with the `exaDEM` data structure.

The tangential force represents the frictional resistance between particles when they slide against each other. This force is calculated based on the relative tangential velocity (:math:`v_t`) and a tangential stiffness parameter (:math:`k_t`, called ``ktContact`` in Rockable).

The **Coulomb friction model** is used to limit the tangential force, ensuring that it does not exceed the product of the friction coefficient :math:`\mu` and the normal force (:math:`f_n`):

.. math::

    f_t \leq \mu \cdot f_n

The tangential force :math:`f_t` is incrementally updated at each time step according to the following relation, starting from the onset of contact:

.. math::

    f_t = f_t + k_t \cdot v_t \cdot \Delta t

where:

- :math:`f_t` is the tangential force,
- :math:`k_t` is the tangential stiffness,
- :math:`v_t` is the relative tangential velocity, and
- :math:`\delta t` is the time step.

The friction force is reset to zero as soon as contact is lost.

``cohesive`` law: cohesive normal force
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
This interaction requires two additional parameters:

.. list-table::
   :widths: 25 75
   :header-rows: 1

   * - Constant
     - Description
   * - :math:`fc`
     - Cohesive force threshold
   * - :math:`dncut`
     - Distance cutoff for cohesive interaction

There are three cases, depending on the interpenetration :math:`\delta_n` between the two particles:

* :math:`\delta_n < -dncut`
* :math:`-dncut < \delta_n < 0`
* :math:`0 < \delta_n < dncut`

.. warning::

  Cohesive forces (`dncut`) are only applied if you use the operators ``contact_[*]_[*]_[*]_cohesive``. Otherwise we only consider the case :math:`\delta_n < 0.0`.

**Formula between particle i and particle j if** :math:`\delta_n < -dncut` **:**


.. math::

  \textbf{f}_{ij} =  -k_n . \delta_n + \alpha_n \sqrt{2.m_{eff}} v_n + k_t . v_t . \Delta_t

with the effective mass:

.. math::

  m_{eff} = \frac{m_i.m_j}{m_i+m_j}

and the relative velocity norm:

.. math::

  v_n = (v_i - (cp - r_i) \wedge vrot_i) - (v_j - (cp - r_j) \wedge vrot_j) 

**Formula between particle i and particle j if** :math:`-dncut < \delta_n < 0` **:**

.. math::

  \textbf{f}_{ij} = f_n + k_t . v_t . \Delta_t

with:

.. math::

   f_n =\left \{
   \begin{array}{lcl}
   -k_n . \delta_n + \alpha_n \sqrt{2.m_{eff}} v_n  &  if  & -k_n . \delta_n + \alpha_n \sqrt{2.m_{eff}} v_n >= -fc \\
   & & \\
   -fc & if  & -k_n . \delta_n + \alpha_n \sqrt{2.m_{eff}} v_n < -fc 
   \end{array} 
   \right.

**Formula between particle i and particle j if** :math:`0 < \delta_n < dncut` **:**

.. math::

  \textbf{f}_{ij} = (\frac{fc}{dncut} . \delta_n - fc) . \textbf{n}

with **n** the normalized vector from particle i to particle j


``dmt`` law: adhesive normal force (Van der Waals)
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
DMT (Derjaguin–Muller–Toporov) is usually used to compute pull-off forces (the force needed to split two rigid objects in contact), computed from a global energy balance.
To define it, it needs an additional parameter representing an energy:

.. list-table::
   :widths: 25 75
   :header-rows: 1

   * - Constant
     - Description
   * - :math:`\gamma`
     - Adhesion energy per unit of surface

In case of contact, an additional force :math:`f_{DMT}` is added to the normal force such that :

.. math::
  f_n =  f_n - f_{DMT}

with:

.. math::
  f_{DMT} = 2 \pi R_{eff} \gamma

where the effective radius :math:`R_{eff}` is obtained from the respective radii :math:`R_{i}` and :math:`R_{j}` of the particles i and j:

.. math::
  R_{eff} = \frac{R_i.R_j}{R_i+R_j}

and :math:`\gamma` is the surface energy (linked to Van der Waals forces).

.. note::
   Since DMT force law is added to the normal force, it requires the definition of a pure contact law, such as ``hooke`` one.


.. _force_field_contact_law_operators:

Contact Law Operators
---------------------

Contact Law Operators select a contact law and define its parameters.

The operator naming convention follows the rule below:

::

   contact_[material_mode]_[particle_type]_[pure_contact_law]_[cohesion_law]

Where each field is defined as follows:

.. list-table:: Operator name fields
   :header-rows: 1
   :widths: 25 25 50

   * - Field
     - Possible values
     - Description
   * - [material_mode]
     - ``singlemat``, ``multimat``
     - Material interaction mode
   * - [particle_type]
     - ``sphere``, ``polyhedron``
     - Type of particles
   * - [pure_contact_law]
     - ``hooke``
     - Pure contact law
   * - [cohesion_law]
     - ``cohesive``, ``dmt``, ``none``
     - Cohesion / adhesion law

.. note::

   For convenience, simplified operator names are available.
   They are strict equivalents of the full naming convention.

   **Sphere operators**

   - ``contact_sphere``  
     is equivalent to  
     ``contact_singlemat_sphere_hooke`` or  
     ``contact_singlemat_sphere_hooke_none``

   - ``contact_multimat_sphere``  
     is equivalent to  
     ``contact_multimat_sphere_hooke_none``

   **Polyhedron operators**

   - ``contact_polyhedron``  
     is equivalent to  
     ``contact_singlemat_polyhedron_hooke`` or  
     ``contact_singlemat_polyhedron_hooke_none``

   - ``contact_multimat_polyhedron``  
     is equivalent to  
     ``contact_multimat_polyhedron_hooke_none``

In a contact operator, the following parameters can be defined:

+---------------------+------------------------------------------------------------------------------+
| `symetric`          | Activate or disable symmetric updates (do not disable it with polyhedron).   |
+---------------------+------------------------------------------------------------------------------+
| `config`            | Data structure that contains contact force parameters (dncut, kn, kt,        |
|                     | kr, fc, mu, damp_rate). Type = exaDEM::ContactParams. No default parameter.  |
+---------------------+------------------------------------------------------------------------------+
| `config_driver`     | Data structure that contains contact force parameters (dncut, kn, kt,        |
|                     | kr, fc, mu, damp_rate). Type = exaDEM::ContactParams.                        |
|                     | This parameter is optional.                                                  |
+---------------------+------------------------------------------------------------------------------+
| `save_interactions` | Store interactions into the classifier data structure. Default is false.     |
+---------------------+------------------------------------------------------------------------------+

Here are four YAML examples:

.. code-block:: yaml

   - contact_sphere:
      symetric: true
      config: { kn: 100000, kt: 100000, kr: 0.1, mu: 0.9, damp_rate: 0.9}

.. code-block:: yaml

   - contact_polyhedron:
      config: { kn: 10000, kt: 10000, kr: 0.1, mu: 0.1, damp_rate: 0.9}
      config_driver: { kn: 10000, kt: 10000, kr: 0.1, mu: 0.3, damp_rate: 0.9}

.. code-block:: yaml

   - contact_singlemat_sphere_hooke_cohesive:
      symetric: true
      config: { dncut: 0.1 m, kn: 100000, kt: 100000, kr: 0.1, fc: 0.05, mu: 0.9, damp_rate: 0.9}

.. code-block:: yaml

   - contact_singlemat_polyhedron_hooke_cohesive:
      config: { dncut: 0.1 m, kn: 10000, kt: 10000, kr: 0.1, fc: 0.05, mu: 0.1, damp_rate: 0.9}
      config_driver: { dncut: 0.1 m, kn: 10000, kt: 10000, kr: 0.1, fc: 0.05, mu: 0.3, damp_rate: 0.9}

.. code-block:: yaml

  - contact_singlemat_sphere_hooke_dmt:
     config: { kn: 1000, kt: 1000, kr: 0.1,mu: 0.2, damp_rate: 0.9, gamma: 10.}
     config_driver: { kn: 1000, kt: 800, kr: 0.1,mu: 0.5, damp_rate: 0.9, gamma: 10.}


.. note::

  If you set ``symetric: false``, make sure the interaction lists were also built without the symmetry option -- by default, ``exaDEM`` always builds them with symmetry enabled, to limit the number of calculations.

.. note::

  The cohesion law ([cohesion_law] = ``cohesive``) adds a cohesion force between `rcut` and `rcut+dncut`, scaled by the cohesion force parameter `fc`.

.. note::

  - The ``contact_[*]_sphere_[*]_[*]``  operators are designed to process interactions built in ``nbh_sphere`` (please, include the config_spheres.msp file).
  - The ``contact_[*]_polyhedron_[*]_[*]`` operators are designed to process interactions built in ``nbh_polyhedron`` (please, include the config_polyhedra.msp file). For the list of polyhedron interaction types these operators process, see :ref:`interaction_type_poly` on the R-Shape / Polyhedron page.


Multi-Material
--------------

In the previous section, the contact law used the same parameters for every interaction (``singlemat`` mode). It is also possible to make the contact law depend on the group of the interacting particles (``multimat`` mode). This section explains how to define contact-law values between particles, as well as between particles and drivers.

.. note::

   Unlike in some other DEM codes, the coefficients are **not** derived from the material
   properties (such as Poisson’s ratio and Young’s modulus).

.. note::

   Since ``exaDEM-1.2.3``, multi-material contact parameters are indexed by ``group`` (an integer, see the ``group`` particle field in the Particle Fields page) instead of directly by particle ``type``. Before ``1.2.3``, the particle ``type`` was used directly. This decouples the shape/material identity (``type``) from the contact-law identity (``group``): several particle types can share the same group and therefore reuse the same contact parameters. Groups must be assigned beforehand, either with the ``group`` parameter of ``set_fields`` or with the dedicated ``set_group`` operator (see the Particle Fields page).

To handle multiple particle groups, use one of the ``contact_multimat_[*]_[*]_[*]`` operators, as illustrated below.

* **YAML example for polyhedra:**

.. code-block:: yaml

   compute_force:
     - gravity_force
     - contact_multimat_polyhedron

* **YAML example for spheres:**

.. code-block:: yaml

   compute_force:
     - gravity_force
     - contact_multimat_sphere:
        symetric: true

The following examples illustrate the definition of contact parameters for two particle
groups (**group 0**, **group 1**) and a driver identified by **0**.

Particle-Particle Contact Parameters
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

* **Operator Name:** ``multimat_contact_params``
* **Description:** This operator defines the contact law parameters between different particle groups.

+--------------------+---------------------------------------------------------------+
| **Parameter**      | **Description**                                               |
+====================+===============================================================+
| ``group1``         | List of the first particle group indice(s).                   |
+--------------------+---------------------------------------------------------------+
| ``group2``         | List of the second particle group indice(s).                  |
+--------------------+---------------------------------------------------------------+
| ``kn``             | Normal force coefficient for the specified interaction type.  |
+--------------------+---------------------------------------------------------------+
| ``kt``             | Tangential force coefficient for the specified interaction    |
|                    | type.                                                         |
+--------------------+---------------------------------------------------------------+
| ``kr``             | Rolling resistance coefficient for the specified              |
|                    | interaction type.                                             |
+--------------------+---------------------------------------------------------------+
| ``mu``             | Friction coefficient for the specified interaction type.      |
+--------------------+---------------------------------------------------------------+
| ``damprate``       | Damping rate coefficient for the specified interaction type.  |
+--------------------+---------------------------------------------------------------+
| ``default_config`` | Applies the same parameter set to all undefined               |
|                    | interaction configurations.                                   |
+--------------------+---------------------------------------------------------------+
| ``n_groups``       | Number of distinct groups (``INPUT_OUTPUT``, since            |
|                    | ``1.2.3``). Usually left unset: it is then deduced from       |
|                    | ``group1``/``group2`` (max index + 1). If it was already set  |
|                    | upstream (e.g. by ``set_group``), it is checked against       |
|                    | ``group1``/``group2`` instead, and a ``default_config`` must  |
|                    | be provided to cover any group pair not listed explicitly.    |
+--------------------+---------------------------------------------------------------+

.. note::

   ``group1``/``group2`` replace the former ``mat1``/``mat2`` parameters (pre-``1.2.3``, which took particle type names directly). Particle groups must be assigned beforehand via the ``group`` parameter of ``set_fields`` or the ``set_group`` operator.

YAML example:

.. code-block:: yaml

  - multimat_contact_params:
     group1:    [      0,     0,     1 ]
     group2:    [      0,     1,     1 ]
     kn:        [   5000, 10000, 15000 ]
     kt:        [   4000,  8000, 12000 ]
     kr:        [    0.0,   0.0,   0.0 ]
     mu:        [    0.1,   0.2,   0.3 ]
     damprate:  [  0.999, 0.999, 0.999 ]


With `default_config`:

.. code-block:: yaml

  - multimat_contact_params:
     group1:    [      0 ]
     group2:    [      0 ]
     kn:        [   5000 ]
     kt:        [   4000 ]
     kr:        [    0.0 ]
     mu:        [    0.1 ]
     damprate:  [  0.999 ]
     default_config: { kn: 15000, kt: 12000, kr: 0.0, mu: 0.3, damp_rate: 0.999 }

A complete example is available (please report if the link does not work): `rotating-multimat.msp <https://github.com/Collab4exaNBody/exaDEM/blob/main/example/polyhedra/multimat/rotating-multimat.msp>`_

Particle-Driver Contact Parameters
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

* **Operator Name:** ``drivers_contact_params``
* **Description:** This operator defines the contact law parameters between particles and drivers.

* **Parameters:**

+--------------------+---------------------------------------------------------------+
| **Parameter**      | **Description**                                               |
+====================+===============================================================+
| ``group``          | List of particle group indice(s) concerned by the interaction.|
+--------------------+---------------------------------------------------------------+
| ``driver_id``      | Identifier(s) of the driver(s) interacting with the particles.|
+--------------------+---------------------------------------------------------------+
| ``kn``             | Normal force coefficient for the specified interaction type.  |
+--------------------+---------------------------------------------------------------+
| ``kt``             | Tangential force coefficient for the specified interaction    |
|                    | type.                                                         |
+--------------------+---------------------------------------------------------------+
| ``kr``             | Rolling resistance coefficient for the specified              |
|                    | interaction type.                                             |
+--------------------+---------------------------------------------------------------+
| ``mu``             | Friction coefficient for the specified interaction type.      |
+--------------------+---------------------------------------------------------------+
| ``damprate``       | Damping rate coefficient for the specified interaction type.  |
+--------------------+---------------------------------------------------------------+
| ``default_config`` | Applies the same parameter set to all undefined               |
|                    | interaction configurations.                                   |
+--------------------+---------------------------------------------------------------+

.. note::

   ``group`` replaces the former ``mat`` parameter (pre-``1.2.3``, which took particle type names directly). Particle groups must be assigned beforehand via the ``group`` parameter of ``set_fields`` or the ``set_group`` operator.

.. code-block:: yaml

  - drivers_contact_params:
     group:     [      0,     1 ]
     driver_id: [      0,     0 ]
     kn:        [  10000, 15000 ]
     kt:        [   8000, 12000 ]
     kr:        [    0.0,   0.1 ]
     mu:        [    0.5,   0.5 ]
     damprate:  [  0.999, 0.999 ]

A complete example is available (please report if the link does not work): `rotating-multimat.msp <https://github.com/Collab4exaNBody/exaDEM/blob/main/example/polyhedra/multimat/rotating-multimat.msp>`_

Checking Group Completeness
~~~~~~~~~~~~~~~~~~~~~~~~~~~

* **Operator Name:** ``check_group_completeness`` (since ``exaDEM-1.2.3``)
* **Description:** Verifies that the number of groups configured in ``multimat_cp`` (via ``multimat_contact_params`` and/or ``drivers_contact_params``) is large enough to cover the maximum ``group`` index actually present on particles. The simulation stops if a particle references a group for which no contact parameters were set up.
* **Parameters:** none.

.. code-block:: yaml

  - check_group_completeness

.. note::

   This operator has no effect (it silently returns) if ``multimat_cp`` has not been produced yet, so it can safely be placed anywhere after ``multimat_contact_params`` and/or ``drivers_contact_params`` in the operator chain, typically right before ``compute_force``.

External Forces
---------------

External forces are additional influences acting on particles within the simulation environment, originating from sources outside the particle system itself. These forces can include environmental factors like wind, fluid flow, or magnetic fields, as well as user-defined forces applied to specific particles or regions.

Gravity Operator
~~~~~~~~~~~~~~~~

Formula:

.. math::
   :label: eqgravity

   \textbf{f} = m.\textbf{g}  

With **f** the forces, m the particle mass, and **g** the gravity constant.

* Operator Name: ``gravity_force``
* Description: This operator computes forces related to the gravity. 
* Parameter:

+-----------+----------------------------------------------------------------------------------------------------------------------------+
| `gravity` |  Gravity vector, one value per axis. Default values are x = 0, y = 0, z = -9.807                                           |
+-----------+----------------------------------------------------------------------------------------------------------------------------+

``YAML`` example:

.. code-block:: yaml

   - gravity_force:
      gravity: [0,0,-0.009807]


Quadratic Drag Force
~~~~~~~~~~~~~~~~~~~~

Formula:

.. math::

   \textbf{f} = -\mu.cx.\|v\|.\textbf{v}  

With **f** the particle force, ``cx`` the aerodynamic coefficient, :math:`\mu` the drag coefficient, :math:`\|v\|` the norm of the particle velocity, and **v** the particle velocity.

* Operator Name: ``quadratic_force``
* Description: External force modeling air or fluid drag: :math:`\textbf{f} = -\mu \cdot cx \cdot \|v\| \cdot \textbf{v}`.
* Parameter:

+------+------------------------------------------------------------+
| `cx` |  aerodynamic coefficient, default value is for air = 0.38. |
+------+------------------------------------------------------------+
| `mu` | drag coefficient. default value is for air = 0.000015.     |
+------+------------------------------------------------------------+


``YAML`` example: see example `quadratic-force-test/QuadraticForceInput.msp`

.. code-block:: yaml

   - quadratic_force:
      cx: 0.38
      mu: 0.0000015

Fluid Grid Force
~~~~~~~~~~~~~~~~

* Operator Name: ``sphere_fluid_friction``
* Description: External force modeling drag from a fluid velocity field defined on a grid.

.. math::

  dv = fv - pv

.. math::

  f = cx . dv . ||dv|| . \pi . r . r.

With `fv` the fluid velocity, `pv` the particle velocity, `r` the particle radius, and `cx` a coefficient set to 1 by default.

.. note::

  The fluid velocity `fv` for each point of the grid has been defined by the operator `set_cell_values` (pure exaNBody operator).

.. _force_field_inner_bond_forces:

Inner Bond Forces
-----------------

.. note::

   This section describes the *force law* applied between already-bonded faces. The
   geometric/sticking side of fragmentation -- pre-cutting grains, the sticking threshold that
   decides which vertices get bonded in the first place, and interface breakage -- is described
   in the Fragmentation Feature section of the R-Shape / Polyhedron page (see
   :ref:`polyhedra_fragmentation`).

The ``inner_bond_force`` law models a **cohesive bond** between two contacting
polyhedron faces, combining a linear elastic + viscous **normal** force, a
linear elastic **tangential** force accumulated over the contact history, and
a **fracture criterion** that breaks the bond once the stored elastic energy
exceeds a material-dependent threshold.

With overlap :math:`\delta = d_n - d_{n0}` and relative normal velocity :math:`v_n`:

.. math::

   f_n &= -k_n\, w\, \delta + \gamma\, v_n \\
   f_t &= w\, k_t\, \mathbf{t_{ds}}

where :math:`w` is the per-interaction weight and :math:`\gamma` is the
viscous damping coefficient derived from ``damp_rate``. The stored elastic
energies (tension only) are:

.. math::

   E_n &= \tfrac12\, w\, k_n\, \delta^2 \\
   E_t &= \tfrac12\, w\, k_t\, \lVert \mathbf{t_{ds}} \rVert^2

Inner Bond Parameters
~~~~~~~~~~~~~~~~~~~~~

.. list-table::
   :header-rows: 1
   :widths: 12 10 78

   * - Name
     - Unit
     - Description
   * - ``kn``
     - force/length
     - Normal stiffness coefficient.
   * - ``kt``
     - force/length
     - Tangential stiffness coefficient.
   * - ``damp_rate``
     - dimensionless
     - Normal damping ratio.
   * - ``g``
     - energy/area
     - **MixedMode** fracture energy release rate. Mutually exclusive with
       ``gn``/``gt``. Breaks when :math:`E_n + E_t > 2\,A\,g`.
   * - ``gn``
     - energy/area
     - **SeparateModes** normal fracture energy release rate (requires ``gt``).
       Breaks when :math:`E_n > 2\,A\,g_n`.
   * - ``gt``
     - energy/area
     - **SeparateModes** tangential fracture energy release rate (requires ``gn``).
       Breaks when :math:`E_t > 2\,A\,g_t`.

.. note::

   Exactly one fracture mode must be configured: ``g`` alone (MixedMode), or
   ``gn`` together with ``gt`` (SeparateModes). Mixing the two, or omitting
   both, is rejected at parsing time.

Usage Examples
~~~~~~~~~~~~~~

Single configuration (no groups), mixed-mode fracture:

.. code-block:: yaml

   inner_bond_params:
     kn: 10000 N/m
     kt: 8000 N/m
     damp_rate: 0.1
     g: 5.0 J/m^2

Single configuration (no groups), separate-mode fracture:

.. code-block:: yaml

   inner_bond_params:
     kn: 10000 N/m
     kt: 8000 N/m
     damp_rate: 0.1
     gn: 5.0 J/m^2
     gt: 2.0 J/m^2

With multiple groups, use the ``inner_bond_params`` operator with one entry
per ``(group1[p], group2[p])`` pair in parallel arrays:

.. note::

   Since ``exaDEM-1.2.3``, pairs are defined with ``group1``/``group2`` (particle group indices) instead of the former ``mat1``/``mat2`` (particle type names). Particle groups must be assigned beforehand via the ``group`` parameter of ``set_fields`` or the ``set_group`` operator.

.. code-block:: yaml

   - inner_bond_params:
      group1:    [      0,     0,     1 ]
      group2:    [      0,     1,     1 ]
      kn:        [   5000, 10000, 15000 ]
      kt:        [   4000,  8000, 12000 ]
      damp_rate: [  0.999, 0.999, 0.999 ]
      gn:        [   1e-5,  1e-5,  1e-5 ]
      gt:        [   1e-5,  1e-5,  1e-5 ]

   # or, mixed mode for every pair:
   - inner_bond_params:
      group1:    [      0,     1 ]
      group2:    [      0,     1 ]
      kn:        [   5000, 15000 ]
      kt:        [   4000, 12000 ]
      damp_rate: [  0.999,  0.999 ]
      g:         [   1e-5,   1e-5 ]

See also
~~~~~~~~

- ``stick_polyhedra`` — builds the inner-bond interactions and computes the
  fracture criterion's area term per bonded face.
- ``apply_interface_fracture_criterion`` — applies the configured rupture
  mode (mixed or separate) to decide whether a bonded interface breaks.

