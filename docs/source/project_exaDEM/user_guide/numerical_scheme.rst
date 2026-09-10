Numerical Scheme
================

.. |dt| replace:: :math:`\Delta_t`
.. |bq| replace:: :math:`\bar{Q}`
.. |do| replace:: :math:`\dot{\omega}`

Velocity Verlet Scheme
^^^^^^^^^^^^^^^^^^^^^^

The time integration in ``exaDEM`` uses the velocity form of the Störmer-Verlet algorithm, better known as *velocity-Verlet*. It is well suited to DEM simulations for its simplicity, stability, and accuracy, providing a straightforward way to integrate Newton's equations of motion and compute positions and velocities efficiently. At each time step, it proceeds as follows:

1. Calculate the position vector at full timestep:

.. math::

    \mathbf{x} \left( t + \Delta t \right) = \mathbf{x} \left( t \right) + \mathbf{v} \left( t \right) \Delta t + \mathbf{a} \left(t\right)\frac{\Delta t}{2}

2. Calculate the velocity vector at half timestep:

.. math::

    \mathbf{v} \left( t + \frac{\Delta t}{2} \right) = \mathbf{v} \left( t \right) + \mathbf{a} \left( t \right) \frac{\Delta t}{2}
   

3. Compute the acceleration vector at full time step, :math:`\mathbf{a} \left( t + \Delta t\right)`, from the interatomic potential, using the position at full time step, :math:`\mathbf{x} \left( t + \Delta t\right)`

4. Finally, calculate the velocity vector at full timestep:
   
.. math::

    \mathbf{v} \left( t + \Delta t \right) = \mathbf{v} \left( t + \frac{\Delta t}{2} \right) + \frac{1}{2} \mathbf{a} \left( t + \Delta t\right) \Delta t

In ``exaDEM``, the numerical scheme definition can be found in ``exaDEM/data/config/config_numerical_schemes.msp``. Written out step by step (without the field-loading optimizations described below), the Velocity-Verlet scheme reads:

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
      - compute_force
      - force_to_accel
      - push_to_angular_acceleration
      - push_to_angular_velocity: { dt_scale: 1 }
      - push_f_v: { dt_scale: 0.5 }

.. note::

  To avoid loading the same fields several times, some of these operators are in practice
  combined into ``combined_compute_prolog`` and ``combined_compute_epilog`` (see below), and the
  default config additionally interleaves driver-specific counterparts of several of these steps
  (e.g. ``push_f_v_r_driver``, ``force_to_accel_driver``). The above is the conceptual,
  fully-expanded form; see ``config_numerical_schemes.msp`` itself for the exact, up-to-date
  default pipeline.

The ``exaNBody`` code provides a generic operator for 1st order time-integration purposes. For example, the file ``exaNBody/src/exanb/push_vec3_1st_order_xform.cpp`` provides 3 different variants:

- ``push_v_r`` : for updating positions from velocities
- ``push_f_v`` : for updating velocities from forces (i.e. acceleration)
- ``push_f_r`` : for updating positions from forces (i.e. acceleration)

In addition, the ``exaNBody`` code also provides a generic operator for 2nd order time-integration purposes. For example, the file ``exaNBody/src/exanb/push_vec3_2nd_order_xform.cpp`` provides the following variant:

- ``push_f_v_r`` : for updating positions from both velocities and forces (i.e. accelerations)

Since in ``exaDEM`` positions are expressed in a reduced frame, the argument ``xform_mode: INV_XFORM`` is mandatory when using any operator that updates the particles positions. The operators are described in detail in the following section.

Operators
^^^^^^^^^

This section describes each operator individually, so you can reorder, add, or remove them to build a time-integration scheme other than Velocity-Verlet. For performance reasons, some of them are merged into a single operator, such as ``combined_compute_epilog`` and ``combined_compute_prolog``.


Reset Forces and Moments
------------------------

* Operator Name: ``reset_force_moment``
* Description: This operator resets two grid fields: moments and forces.

Here is a YAML example:

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

Here is a YAML example:

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

Here is a YAML example:

.. code-block:: yaml

  - push_to_quaternion: { dt_scale: 1.0 }


Update Angular Velocity
-----------------------

* Operator Name: ``push_to_angular_velocity``
* Description: This operator computes particle angular velocity values from angular velocities and angular accelerations. 
* Parameter:

  * ``dt_scale``: Coefficient applied to the increment time (|dt|) 

Formula:

.. math::

  av = av + aa.\frac{\Delta_t^2}{2}

with **aa** the angular acceleration, **av** the angular velocity, and Q the particle orientation. 

Here is a YAML example:

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

with **aa** the angular acceleration, **av** the angular velocity, I the particle inertia, and Q the particle orientation (and |bq| its conjugate). To compute |do|, we need the particle moment and the particle inertia values.

Here is a YAML example:

.. code-block:: yaml

  - push_to_angular_acceleration

.. note::

  This operator is not (directly) used, it has been merged in the operator ``combined_compute_epilog`` 

Combined Prolog
---------------

* Operator Name: ``combined_compute_prolog``
* Description: This is an operator that combines 3 operators, in order, with the same
  ``dt_scale`` values as steps 1, 2, and 3 of the scheme above (1.0, 0.5, and 1.0
  respectively -- not user-configurable, unlike the standalone operators):

  * ``push_f_v_r``
  * ``push_f_v``
  * ``push_to_quaternion``

Here is a YAML example:

.. code-block:: yaml

  - combined_compute_prolog  

Combined Epilog
---------------

* Operator Name: ``combined_compute_epilog``
* Description: This is an operator that combines 3 operators, in order:

  * ``push_to_angular_acceleration``
  * ``push_to_angular_velocity``
  * ``push_f_v``

Here is a YAML example:

.. code-block:: yaml

  - combined_compute_epilog 




