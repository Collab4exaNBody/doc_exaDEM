Numerical Scheme
================

.. |dt| replace:: :math:`\Delta_t`

Velocity Verlet Scheme
^^^^^^^^^^^^^^^^^^^^^^

The time integration in ``exaDEM`` is performed using the velocity form of the Störmer-Verlet time integration algorithm, well-known as the `velocity-Verlet` algorithm. It is advantageous for DEM simulations due to its simplicity, stability, and accuracy. It provides a straightforward method for integrating Newton's equations of motion, efficiently calculating both positions and velocities. The `velocity-verlet` algorithm is integrated using the following scheme at each time step:

1. Calculate position vector at full time-step:

.. math::

    \mathbf{x} \left( t + \Delta t \right) = \mathbf{x} \left( t \right) + \mathbf{v} \left( t \right) \Delta t + \mathbf{a} \left(t\right)\frac{\Delta t}{2}

2. Calculate velocity vector at half time-step:

.. math::

    \mathbf{v} \left( t + \frac{\Delta t}{2} \right) = \mathbf{v} \left( t \right) + \mathbf{a} \left( t \right) \frac{\Delta t}{2}
   

3. Compute the acceleration vector at full time-step \\( \\mathbf{a} \\left( t + \\Delta t\\right) \\) from the interatomic potential using the position at full time-step \\( \\mathbf{x} \\left( t + \\Delta t\\right) \\)

4. Finally, calculate the velocity vector at full time-step:
   
.. math::

    \mathbf{v} \left( t + \Delta t \right) = \mathbf{v} \left( t + \frac{\Delta t}{2} \right) + \frac{1}{2} \mathbf{a} \left( t + \Delta t\right) \Delta t

In ``exaDEM``, the numerical scheme definition can be found in ``exaDEM/data/config/config_numerical_schemes.msp`` and the YAML block for the Velocity-Verlet scheme reads

.. code-block:: yaml

   numerical_scheme: numerical_scheme_verlet
   
   numerical_scheme_verlet:
     name: scheme
     body:
      - push_f_v_r: { dt_scale: 1.0 }
      - push_f_v: { dt_scale: 0.5 }
      - push_to_quaternion: { dt_scale: 1.0 }
      - check_and_update_particles
      - reset_force_moment
      - compute_force_op
      - force_to_accel
      - update_angular_acceleration 
      - update_angular_velocity: { dt_scale: 1 }
      - push_f_v: { dt_scale: 0.5 }

.. note::

  Some operators are combined in operators to optimize code and avoid loading the same fields several times: ``combined_compute_prolog`` and ``combined_compute_epilog``. 

The ``exaNBody`` code provides a generic operator for 1st order time-integration purposes. For example, the file ``exaNBody/src/exanb/push_vec3_1st_order_xform.cpp`` provides 3 different variants:

- ``push_v_r`` : for updating positions from velocities
- ``push_f_v`` : for updating velocities from forces (i.e. acceleration)
- ``push_f_r`` : for udpdating positions from forces (i.e. acceleration)

In addition, the ``exaNBody`` code also provides a generic operator for 2nd order time-integration purposes. For example, the file ``exaNBody/src/exanb/push_vec3_2nd_order_xform.cpp`` provides the following variant:

- ``push_f_v_r`` : for updating positions from both velocities and forces (i.e. accelerations)

Since in ``exaDEM`` positions are expressed in a reduced frame, the argument ``xform_mode: INV_XFORM`` is mandatory when using any operator that updates the particles positions. The operators are described in detail in the following section.


Operators
^^^^^^^^^

In this section, we'll explain operators and how to use them. The idea is to be able to easily reorder / add / remove operators to easily build another time integration scheme than the Verlet Vitesse scheme. For performance reasons, some operators are merged into a single operator, such as ``combined_compute_epilog`` and ``combined_compute_prolog``.


Reset Forces and Moments
------------------------

* Operator Name: ``reset_force_moment``
* Description: This operator resets two grid fields : moments and forces.

Here a YAML example:

.. code-block:: yaml

  - reset_force_moment

Update Particle Acceleration
----------------------------

* Operator Name: ``force_to_accel``
* Description: This operator computes particle accelerations from forces and mass.

Formula:

.. math::

  f = a.m;

with **f** the sum of forces applied to the particle, **a** the acceleration of the particle, and m its mass. 

Here a YAML example:

.. code-block:: yaml

  - force_to_accel


Update Particle Orientation
---------------------------

* Operator Name: ``push_to_quaternion``
* Description: This operator computes particle orientations from angular velocities and angular accelerations. 
* Parameter:

  * ``dt_scale``: Coefficient applied to the increment time (|dt|) 

Formula:

.. math::

  Q = Q+Q.av.\Delta_t

.. math::

  Q = \frac{Q}{||Q||}

.. math::

  av = av + aa.\frac{\Delta_t^2}{2}

with **aa** the angular acceleration, **av** the angular velocity, and Q the particle orientation. 

Here a YAML example:

.. code-block:: yaml

  - push_to_quaternion: { dt_scale: 1.0 }


Update Angular Velocity
-----------------------

* Operator Name: ``push_to_angular_velocity``
* Description: This operator computes particle angular velocitiy values from angular velocities and angular accelerations. 
* Parameter:

  * ``dt_scale``: Coefficient applied to the increment time (|dt|) 

Formula:

.. math::

  av = av + aa.\frac{\Delta_t^2}{2}

with **aa** the angular acceleration, **av** the angular velocity, and Q the particle orientation. 

Here a YAML example:

.. code-block:: yaml

  - push_to_angular_velocity: { dt_scale: 1.0 }

.. note::

  This operator is not (directly) used, it has been merged in the operator ``combined_compute_epilog`` 

Update Angular Acceleration
---------------------------

* Operator Name: ``push_to_angular_acceleration``
* Description: This operator computes angular accelerations.

Formula:

.. math::

  \omega = \bar{Q}.av

.. math::

  aa = Q.\dot{\omega}

.. math::

.. |bq| replace:: :math:`\bar{Q}`
.. |do| replace:: :math:`\dot{\omega}`  

with **aa** the angular acceleration, **av** the angular velocity, I the particle inertia, and Q the particle orientation (and |bq| its conjugate). To compute |do|, we need the particle moment and the particle inertia values. 

Here a YAML example:

.. code-block:: yaml

  - push_to_angular_acceleration

.. note::

  This operator is not (directly) used, it has been merged in the operator ``combined_compute_epilog`` 

Combined Prolog
---------------

* Operator Name: ``combined_compute_prolog``
* Description: This is an operator that combined 3 operators:

  * push_f_v_r
  * push_f_v
  * push_to_quaternion

* Parameter:

  * ``dt_scale``: Coefficient applied to the increment time (|dt|) 

Here a YAML example:

.. code-block:: yaml

  - combined_compute_prolog  

Combined Epilog
---------------

* Operator Name: ``combined_compute_epilog``
* Description: This is an operator that combined 3 operators:

  * push_to_angular_accelartion
  * push_angular_velocity
  * push_f_v

Here a YAML example:

.. code-block:: yaml

  - combined_compute_epilog 




