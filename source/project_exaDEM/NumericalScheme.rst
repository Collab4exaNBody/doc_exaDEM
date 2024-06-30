Numerical Scheme
================


.. |dt| replace:: :math:`\Delta_t`

Velocity Verlet Scheme
^^^^^^^^^^^^^^^^^^^^^^

Time Scheme
-----------

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

  f = a / m;

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




