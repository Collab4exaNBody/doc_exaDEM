Force Field
===========

The force field encompasses a broader set of operators and mechanisms responsible for computing forces acting on particles within the particle environment. It includes various types of forces such as gravitational forces, contact forces, and other external influences that affect particle dynamics.

Contact Forces
--------------

Contact forces specifically refer to interactions between particles or entities that come into direct contact with each other within the simulation. These forces arise from physical contact and can include repulsive forces to prevent particle overlap, frictional forces, and cohesive forces between bonded particles.


Hooke's Law Operators
^^^^^^^^^^^^^^^^^^^^^

``Hooke's Law`` in the context of Discrete Element Method (DEM) refers to the principle used to calculate forces between particles based on their relative displacements. In DEM simulations, ``Hooke's Law`` is applied to model ``interactions`` between particles, enabling the simulation of elastic deformation and linear force behaviors within particle-based systems.


Variables:

* \\(cp\\) : The contact position
* \\(r_i\\) : The position of the particle i
* \\(r_j\\) : The position of the particle j
* \\(v_i\\) : The velocity of the particle i
* \\(v_j\\) : The velocity of the particle j
* \\(vrot_i\\) : The angular velocity of the particle i
* \\(vrot_j\\) : The angular velocity of the particle j
* \\(m_i\\) : The mass of the particle i
* \\(m_j\\) : The mass of the particle j
* \\(d_n\\) : The interpenetration / particle overlap

And constants:

* \\(\\Delta_t\\) : The timestep increment
* \\(\\alpha_n\\) : The damping rate
* \\(k_n\\) : The normal stiffness coefficient
* \\(k_t\\) : The tangential stiffness coefficient
* \\(v_t\\) : The relative tangential velocity 
* \\(k_r\\) : The rotational stiffness coefficient
* \\(\mu\\) : The coefficient of friction
* \\(fc\\) : The cohesive coefficient
* \\(dncut\\) : 


Three possibilities depending on the value of the interpenetration between \\(d_n\\) two particles:

*  \\( d_n < -dncut \\)
*  \\( -dncut < d_n < 0 \\)
*  \\( 0 < d_n < dncut \\)

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

* Name: ``hooke_sphere`` or ``hooke_polyehdron``
* Description: These operators compute forces between particles and particles/drivers using the Hooke's law.
* Paremeter:

  * `symetric`: Activate or disable symetric updates (do not disable it with polyhedron).
  * `config`:  Data structure that contains hooke force parameters (rcut, dncut, kn, kt, kr, fc, mu, damp_rate). Type = exaDEM::HookeParams. No default parameter.
  * `config_driver`:  Data structure that contains hooke force parameters (rcut, dncut, kn, kt, kr, fc, mu, damp_rate). Type = exaDEM::HookeParams. This parameter is optional.

Here two examples with YAML:

.. code-block:: yaml

   - hooke_sphere:
      config: { rcut: 1.1 m , dncut: 1.1 m, kn: 100000, kt: 100000, kr: 0.1, fc: 0.05, mu: 0.9, damp_rate: 0.9}

.. code-block:: yaml

   - hooke_polyhedron:
      config: { rcut: 0.0 m , dncut: 0.0 m, kn: 10000, kt: 10000, kr: 0.1, fc: 0.05, mu: 0.1, damp_rate: 0.9}
      config_driver: { rcut: 0.0 m , dncut: 0.0 m, kn: 10000, kt: 10000, kr: 0.1, fc: 0.05, mu: 0.3, damp_rate: 0.9}

.. note::

  It is important to check that interaction lists have been built with this option enabled. By default, `exaDEM` always builds interaction lists using the symmetry option to limit the number of calculations.

.. note::

  ``Hooke force`` operator includes a cohesion force from `rcut` to `rcut+dncut` with the cohesion force parameter `fc`.

.. note::

  - rcut is not used in the contexte of simulations with polyhedra.
  - This operator is designed to process interactions built in ``nbh_polyhedron`` (spheropolyhedra).

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

  * `gravity`:  Define the gravity constant in function of the gravity axis, default value are x axis = 0, y axis = 0 and z axis = -9.807

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

  * `cx` :  aerodynamic coefficient, default value is for air = 0.38.
  * `mu` : drag coefficient. default value is for air = 0.000015.


``YAML`` example:  see example `quadratic-force-test/QuadraticForceInput.msp`

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

With `fv` the fuild velocity, `pv` the particle velocity, `r` the particle radius and `cx` a coefficient set to 1 by default.

.. note::

  The fluid velocity `fv` for each point of the grid has been defined by the operator `set_cell_values` (pure exaNBody operator)

