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

* Name: ``hooke_force``
* Description: This operator computes forces between spherical particles using the Hooke law. 
* Paremeter:

  * `config`:  Data structure that contains hooke force parameters (rcut, dncut, kn, kt, kr, fc, mu, damp_rate). Type = exaDEM::HookeParams. No default operator.

.. code-block:: yaml

   - hooke_force:
      config: { rcut: 1.1 m , dncut: 1.1 m, kn: 100000, kt: 100000, kr: 0.1, fc: 0.05, mu: 0.9, damp_rate: 0.9}

.. note::

  ``Hooke force`` operator includes a cohesion force from `rcut` to `rcut+dncut` with the cohesion force parameter `fc`.

* Name: ``hooke_force_driver``
* Description: This operator computes forces between spherical particles and drivers using the Hooke law. 
* Paremeter:

  * `config`:  Data structure that contains hooke force parameters (rcut, dncut, kn, kt, kr, fc, mu, damp_rate). Type = exaDEM::HookeParams. No default operator.

.. code-block:: yaml

   - hooke_force_driver:
      config: { rcut: 1.1 m , dncut: 1.1 m, kn: 100000, kt: 100000, kr: 0.1, fc: 0.05, mu: 0.9, damp_rate: 0.9}


.. note::

  ``Hooke force driver`` operator processes contact between spherical particles and ``drivers`` stored into the list of ``drivers`` such as walls/surfaces or balls/spheres. 

.. warning::

  This operator is used with spherical particles.


Other ``hooke forces`` are defined when the data interaction is used and filled via the opertators ``nbh_sphere_sym`` or ``nbh_sphere_no_sym``.

* Operator Name: ``hooke_sphere_sym`` / ``hooke_sphere_no_sym``
* Description: This operator computes forces between spherical particles themselves and with drivers using the Hooke law.
* Parameters:

  * `config`:  Data structure that contains hooke force parameters for interactions between a polyhedron and a polyhedron (rcut, dncut, kn, kt, kr, fc, mu, damp_rate). Type = exaDEM::HookeParams.
  * `config_driver`:  Data structure that contains hooke force parameters for interactions between a polyhedron and a driver (rcut, dncut, kn, kt, kr, fc, mu, damp_rate). Type = exaDEM::HookeParams.

.. code-block:: yaml

   - hooke_sphere_sym:
      config: { rcut: 0.0 m , dncut: 0.0 m, kn: 10000, kt: 10000, kr: 0.1, fc: 0.05, mu: 0.1, damp_rate: 0.9}
      config_driver: { rcut: 0.0 m , dncut: 0.0 m, kn: 10000, kt: 10000, kr: 0.1, fc: 0.05, mu: 0.3, damp_rate: 0.9}

* Operator Name: ``hooke_polyhedron``
* Description: This operator computes forces between polyhedral particles themselves and with drivers using the Hooke law.
* Parameters:

  * `config`:  Data structure that contains hooke force parameters for interactions between a polyhedron and a polyhedron (rcut, dncut, kn, kt, kr, fc, mu, damp_rate). Type = exaDEM::HookeParams.
  * `config_driver`:  Data structure that contains hooke force parameters for interactions between a polyhedron and a driver (rcut, dncut, kn, kt, kr, fc, mu, damp_rate). Type = exaDEM::HookeParams.

``YAML`` example:


.. code-block:: yaml

   - hooke_polyhedron:
      config: { rcut: 0.0 m , dncut: 0.0 m, kn: 10000, kt: 10000, kr: 0.1, fc: 0.05, mu: 0.1, damp_rate: 0.9}
      config_driver: { rcut: 0.0 m , dncut: 0.0 m, kn: 10000, kt: 10000, kr: 0.1, fc: 0.05, mu: 0.3, damp_rate: 0.9}

.. note::

  - rcut is not used in the contexte of simulations with polyhedra.
  - This operator is designed to process interactions built in ``nbh_polyhedron`` (spheropolyhedra).

Hooke Law Sphere - Driver Operators (legacy)
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

.. warning::

  These operators are destined to disappear with the factorization of drivers.

* Name: ``rigid_surface``
* Description: This operator computes forces between particles and a rigid surface (named wall in other operators) using the Hooke law.          
* Parameters:

   * `damprate` : Parameter of the force law used to model contact rigid surface/sphere.
   * `kn` : Parameter of the force law used to model contact rigid surface/sphere.
   * `kr` : Parameter of the force law used to model contact rigid surface/sphere.
   * `kt` : Parameter of the force law used to model contact rigid surface/sphere.
   * `mu` : Parameter of the force law used to model contact rigid surface/sphere.
   * `normal` : Normal vector of the rigid surface. No default value.
   * `offset` : Offset from the origin (0,0,0) of the rigid surface. Default is 0.

``YAML`` example:

.. code-block:: yaml

   - rigid_surface:
      normal: [0,0,1]
      offset: -1
      kt: 80000
      kn: 100000
      kr : 0
      mu: 0.9
      damprate: 0.9

* Operator Name: ``cylinder_wall``
* Description: This operator computes forces for interactions beween particles and a cylinder.
* Parameters:
   * `damprate` : Parameter of the force law used to model contact cylinder/sphere.
   * `kn` : Parameter of the force law used to model contact cylinder/sphere.
   * `kr` : Parameter of the force law used to model contact cylinder/sphere.
   * `kt` : Parameter of the force law used to model contact cylinder/sphere.
   * `mu` : Parameter of the force law used to model contact cylinder/sphere.
   * `radius` : The cylinder radius.
   * `center` : The cylinder center.
   * `axis` : Define the plan of the cylinder
   * `cylinder_angular_velocity` : Angular velocity of the cylinder, default is 0 m.s-1

``YAML`` example:

.. code-block:: yaml

  - cylinder_wall:
     radius: 100
     center: [50,50,50]
     axis: [1,0,1]
     cylinder_angular_velocity: [0,0.017,0]
     kt: 80000
     kn: 100000
     kr : 0
     mu: 0.3
     damprate: 0.9

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

