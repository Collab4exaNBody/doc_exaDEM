Force Field
===========

The force field encompasses a broader set of operators and mechanisms responsible for computing forces acting on particles within the particle environment. It includes various types of forces such as gravitational forces, contact forces, and other external influences that affect particle dynamics.

Contact Force Laws
------------------

In the Discrete Element Method (DEM), the equations of motion (translations and rotations) are discretized in time. Only the rigid-body displacements are considered. Small overlaps between the particles are allowed and used as strain variables. The total contact force between particle :math:`i` and particle :math:`j` is given by 

.. math::

 \textbf{f}_{ij} = f_n \textbf{n}  +  \textbf{f}_t

Twhere :math:`f_n` the normal component of the contact force and :math:`\textbf{f}_t` is the tangential force vector. These forces are expressed in the local contact frame :math:`(\textbf{n},\textbf{t},\textbf{s})` as a function of the overlaps and tangential displacements. They are calculated from force laws that generally describe frictional contact interactions. An important feature of DEM is to allow the particles to overlap. This overlap :math:`\delta_n` represents a normal strain localized in the vicinity of the contact point. A simple linear relation is assumed between normal contact force and :math:`\delta_n` . This is consistent with the fact that the overlaps allow for a penalty-based explicit formulation of particle motions, i.e., the elastic repulsion force is mobilized to prevent penalizing the overlap. The condition of particle undeformability implies that the overlaps must stay small compared to particle size. In this linear approximation, the normal component of the contact force is given by 

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

Tangential Component
--------------------

The tangential force represents the frictional resistance between particles when they slide against each other. This force is calculated based on the relative tangential velocity (:math:`v_t`) and a tangential stiffness parameter (:math:`k_t`, keyword ``ktContact``). 

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

Contact's Law Operators
-----------------------

``Contact's Law`` in the context of the Discrete Element Method (DEM) refers to the principle used to calculate forces between particles based on their relative displacements. In DEM simulations, ``Contact's Law`` is applied to model ``interactions`` between particles, enabling the simulation of elastic deformation and linear force behaviors within particle-based systems.

Variables:

+--------------+-----------------------------------------+
| Variable     | Description                             |
+==============+=========================================+
| \\(cp\\)     | The contact position                    |
+--------------+-----------------------------------------+
| \\(r_i\\)    | The position of the particle i          |
+--------------+-----------------------------------------+
| \\(r_j\\)    | The position of the particle j          |
+--------------+-----------------------------------------+
| \\(v_i\\)    | The velocity of the particle i          |
+--------------+-----------------------------------------+
| \\(v_j\\)    | The velocity of the particle j          |
+--------------+-----------------------------------------+
| \\(vrot_i\\) | The angular velocity of the particle i  |
+--------------+-----------------------------------------+
| \\(vrot_j\\) | The angular velocity of the particle j  |
+--------------+-----------------------------------------+
| \\(m_i\\)    | The mass of the particle i              |
+--------------+-----------------------------------------+
| \\(m_j\\)    | The mass of the particle j              |
+--------------+-----------------------------------------+
| \\(d_n\\)    | The interpenetration / particle overlap |
+--------------+-----------------------------------------+

And constants:

+-----------------+--------------------------------------+
| Constant        | Description                          |
+=================+======================================+
| \\(\\Delta_t\\) | The timestep increment               |
+-----------------+--------------------------------------+
| \\(\\alpha_n\\) | The damping rate                     |
+-----------------+--------------------------------------+
| \\(k_n\\)       | The normal stiffness coefficient     |
+-----------------+--------------------------------------+
| \\(k_t\\)       | The tangential stiffness coefficient |
+-----------------+--------------------------------------+
| \\(v_t\\)       | The relative tangential velocity     |
+-----------------+--------------------------------------+
| \\(k_r\\)       | The rotational stiffness coefficient |
+-----------------+--------------------------------------+
| \\(\mu\\)       | The coefficient of friction          |
+-----------------+--------------------------------------+
| \\(fc\\)        | The cohesive coefficient             |
+-----------------+--------------------------------------+
| \\(dncut\\)     | X                                    |
+-----------------+--------------------------------------+


Three possibilities depending on the value of the interpenetration between \\(d_n\\) two particles:

*  \\( d_n < -dncut \\)
*  \\( -dncut < d_n < 0 \\)
*  \\( 0 < d_n < dncut \\)

.. warning::

  Cohesive forces (`dncut`) are only applied if you use the operators: ``contact_sphere_with_cohesion`` or ``contact_polyhedron_with_cohesion``. Otherwise we only consider the case ::math`d_n < 0.0`.

**Formula between particle i and particle j if \\( d_n < -dncut \\) :**


.. math::

  \textbf{f}_{ij} =  -k_n . d_n + \alpha_n \sqrt{2.m_{eff}} v_n + k_t . v_t . \Delta_t

with the effective mass:

.. math::

  m_{eff} = \frac{m_i.m_j}{m_i+m_j}

and the relative velocity norm:

.. math::

  v_n = (v_i - (cp - r_i) \wedge vrot_i) - (v_j - (cp - r_j) \wedge vrot_j) 

**Formula between particle i and particle j if \\( -dncut < d_n < 0 \\) :**

.. math::

  \textbf{f}_{ij} = f_n + k_t . v_t . \Delta_t

with:

.. math::

   f_n =\left \{
   \begin{array}{lcl}
   -k_n . d_n + \alpha_n \sqrt{2.m_{eff}} v_n  &  if  & -k_n . d_n + \alpha_n \sqrt{2.m_{eff}} v_n >= -fc \\
   & & \\
   -fc & if  & -k_n . d_n + \alpha_n \sqrt{2.m_{eff}} v_n < -fc 
   \end{array} 
   \right.

**Formula between particle i and particle j if \\( 0 < d_n < dncut \\) :**

.. math::

  \textbf{f}_{ij} = (\frac{fc}{dncut} . d_n - fc) . \textbf{n}

with **n** normalized vector from particle i to particle j

* Name: ``contact_sphere``, ``contact_polyehdron``, ``contact_sphere_with_cohesion``, or ``contact_polyehdron_with_cohesion``
* Description: These operators compute forces between particles and particles/drivers using the contact's law.
* Parameter:

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

Here are 4 examples with YAML:

.. code-block:: yaml

   - contact_sphere:
      symetric: true
      config: { kn: 100000, kt: 100000, kr: 0.1, mu: 0.9, damp_rate: 0.9}

.. code-block:: yaml

   - contact_polyhedron:
      config: { kn: 10000, kt: 10000, kr: 0.1, mu: 0.1, damp_rate: 0.9}
      config_driver: { kn: 10000, kt: 10000, kr: 0.1, mu: 0.3, damp_rate: 0.9}

.. code-block:: yaml

   - contact_sphere_with_cohesion:
      symetric: true
      config: { dncut: 0.1 m, kn: 100000, kt: 100000, kr: 0.1, fc: 0.05, mu: 0.9, damp_rate: 0.9}

.. code-block:: yaml

   - contact_polyhedron_with_cohesion:
      config: { dncut: 0.1 m, kn: 10000, kt: 10000, kr: 0.1, fc: 0.05, mu: 0.1, damp_rate: 0.9}
      config_driver: { dncut: 0.1 m, kn: 10000, kt: 10000, kr: 0.1, fc: 0.05, mu: 0.3, damp_rate: 0.9}

.. note::

  It is important to check that interaction lists have been built with this option enabled. By default, `exaDEM` always builds interaction lists using the symmetry option to limit the number of calculations.

.. note::

  ``Contact Force With Cohesion`` operator includes a cohesion force from `rcut` to `rcut+dncut` with the cohesion force parameter `fc`.

.. note::

  - The ``contact_sphere`` and ``contact_sphere_with_cohesion`` operators are designed to process interactions built in ``nbh_sphere`` (please, include the config_spheres.msp file).
  - The ``contact_polyhedron`` and ``contact_polyhedron_with_cohesion`` operators are designed to process interactions built in ``nbh_polyhedron`` (please, include the config_polyhedra.msp file).


Multi-Material
--------------

In the previous section, the contact law used the same parameters for all interactions.  
It is also possible to specify the contact law depending on the type of interaction.  
In this section, we introduce how to define the values of the contact law between particles,  
as well as between particles and drivers.

.. note::

   Unlike in some other DEM codes, the coefficients are **not** derived from the material 
   properties (such as Poisson’s ratio and Young’s modulus).  

To handle multiple particle types, you must use either the ``contact_sphere_multimat`` 
or the ``contact_polyhedron_multimat`` operator, as illustrated below.

* **YAML example for polyhedra:**

.. code-block:: yaml

   compute_force:
     - gravity_force
     - contact_polyhedron_multimat

* **YAML example for spheres:**

.. code-block:: yaml

   compute_force:
     - gravity_force
     - contact_sphere_multimat:
        symetric: true

The following examples illustrate the definition of contact parameters for two particle 
types (**Type1**, **Type2**) and a driver identified by **0**.

Particle-Particle
^^^^^^^^^^^^^^^^^

* **Operator Name:** ``multimat_contact_params``
* **Description:** This operator defines the contact law parameters between different particle types.

+--------------------+---------------------------------------------------------------+
| **Parameter**      | **Description**                                               |
+====================+===============================================================+
| ``mat1``           | List of the first particle type(s).                           |
+--------------------+---------------------------------------------------------------+
| ``mat2``           | List of the second particle type(s).                          |
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

YAML example:

.. code-block:: yaml

  - multimat_contact_params:
     mat1:      [  Type1, Type1, Type2 ]
     mat2:      [  Type1, Type2, Type2 ]
     kn:        [   5000, 10000, 15000 ]
     kt:        [   4000,  8000, 12000 ]
     kr:        [    0.0,   0.0,   0.0 ]
     mu:        [    0.1,   0.2,   0.3 ]
     damprate:  [  0.999, 0.999, 0.999 ]


With `default_config`:

.. code-block:: yaml

  - multimat_contact_params:
     mat1:      [  Type1 ]
     mat2:      [  Type1 ]
     kn:        [   5000 ]
     kt:        [   4000 ]
     kr:        [    0.0 ]
     mu:        [    0.1 ]
     damprate:  [  0.999 ]
     default_config: { kn: 15000, kt: 12000, kr: 0.0, mu: 0.3, damp_rate: 0.999 }

A complete example is available (please report if the link does not work): `rotating-multimat.msp <https://github.com/Collab4exaNBody/exaDEM/blob/main/example/polyhedra/multimat/rotating-multimat.msp>`_

Particle-Driver
^^^^^^^^^^^^^^^

* **Operator Name:** ``drivers_contact_params``
* **Description:** This operator defines the contact law parameters between particles and drivers.

* **Parameters:**

+--------------------+---------------------------------------------------------------+
| **Parameter**      | **Description**                                               |
+====================+===============================================================+
| ``mat``            | List of particle type(s) concerned by the interaction.        |
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

.. code-block:: yaml

  - drivers_contact_params:
     mat:       [  Type1, Type2 ]
     driver_id: [      0,     0 ]
     kn:        [  10000, 15000 ]
     kt:        [   8000, 12000 ]
     kr:        [    0.0,   0.1 ]
     mu:        [    0.5,   0.5 ]
     damprate:  [  0.999, 0.999 ]

A complete example is available (please report if the link does not work): `rotating-multimat.msp <https://github.com/Collab4exaNBody/exaDEM/blob/main/example/polyhedra/multimat/rotating-multimat.msp>`_

External Forces
---------------

External forces are additional influences acting on particles within the simulation environment, originating from sources outside the particle system itself. These forces can include environmental factors like wind, fluid flow, or magnetic fields, as well as user-defined forces applied to specific particles or regions.

Gravity Operator
^^^^^^^^^^^^^^^^

Formula:

.. math::
   :label: eqgravity

   \textbf{f} = m.\textbf{g}  

With **f** the forces, m the particle mass, and **g** the gravity constant.

* Operator Name: ``gravity_force``
* Description: This operator computes forces related to the gravity. 
* Parameter:

+-----------+----------------------------------------------------------------------------------------------------------------------------+
| `gravity` |  Define the gravity constant in function of the gravity axis, default value are x axis = 0, y axis = 0 and z axis = -9.807 |
+-----------+----------------------------------------------------------------------------------------------------------------------------+

``YAML`` example:

.. code-block:: yaml

   - gravity_force:
      gravity: [0,0,-0.009807]


Quadratic Drag Force
^^^^^^^^^^^^^^^^^^^^

Formula:

.. math::

   \textbf{f} = -\mu.cx.\|v\|.\textbf{v}  

With **f** the particle forces, cx the aerodynamic coefficient, and \\(\\mu\\) the drag coefficient, \||v\|| the norm of the particle velocity, and **v** the particle velocity.

* Operator Name: ``quadratic_force``
* Description: External forces that model air or fluid, f = - mu * cx * norm(v) * vector(v).
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
^^^^^^^^^^^^^^^^

* Operator Name: ``sphere_fluid_friction``
* Description: External forces that model a fluid computed from a grid such as: f = ||fv - pv|| 

.. math::

	dv = fv - pv 

.. math::

  f = cx . dv . ||dv|| . \pi . r . r.

With `fv` the fluid velocity, `pv` the particle velocity, `r` the particle radius, and `cx` a coefficient set to 1 by default.

.. note::

  The fluid velocity `fv` for each point of the grid has been defined by the operator `set_cell_values` (pure exaNBody operator).

