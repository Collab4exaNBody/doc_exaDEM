Parallelization
===============

MPI Parallelization
^^^^^^^^^^^^^^^^^^^

Domain Decomposition
--------------------

Cost Model
----------

Thread Parallelization
^^^^^^^^^^^^^^^^^^^^^^

GPU Support
^^^^^^^^^^^

Benchmarks
^^^^^^^^^^

Rotating Drum (Sphere)
----------------------

.. |bench1-picture| image:: ../_static/mpi-dem-example-100M-modified.png
.. |bench1-picture-mpi| image:: ../_static/mpi-dem-example-100M-mpi.png
.. |bench1-graph1| image:: ../_static/drum_dem_100M.png
.. |bench1-graph2| image:: ../_static/drum_dem_100M_comp.png
.. |bench1-graph3| image:: ../_static/drum_dem_100M_comp_pourcentage.png

This benchmark has been presented in 2023 within the paper :cite:`Carrard_2024` : "ExaNBody : a HPC framework for N-Body applications"


Description:

Different OpenMP/Mpi con␜gurations (number of cores/threads per ``mpi`` process) have been tested to balance multi-level parallelism. 
Both simulations were instrumented during 1,000 representative iterations. 
The performance of ``ExaDEM`` was evaluated using up to 256 cluster nodes, built on bi-socket 64-core AMD EPYC ``Milan`` 7763 processors running at 2.45 GHz and equipped with 256 GB of RAM.

.. figure:: ../_static/mpi-dem-example-100M-modified.png
   :scale: 90%
   :align: center

   Dem simulation of 100 million spheres in a rotating drum.


ExaDEM's performance is evaluated with a simulation of a rotating drum containing 100 million spherical particles, see Figure below. 
This setup is a tough benchmark as particles are rapidly moving all around the heterogeneously dense domain, due to gravity. 
Additionally, the employed contact force model has a low arithmetic intensity, and ``exaDEM`` must handle pairwise friction information, that is updated by kernel and must migrate between ``mpi`` processes when subdomains are redistributed. 

.. figure:: ../_static/mpi-dem-example-100M-mpi.png
   :scale: 90%
   :align: center

   Domain decomposition of 100 000 of spheres into a rotating drum

.. figure:: ../_static/drum_dem_100M.png
   :scale: 70%
   :align: center

   Speedup for different OpenMP/Mpi configurations. ExaDEM simulation with 1, 8, and 128 threads per ``mpi`` process.

.. note::

  Note that the ``Milan`` nodes are made up of 128 cores spread over 8 NUMA nodes, and we have pointed out that NUMA effects reduce overall performance.


.. figure:: ../_static/drum_dem_100M_comp.png
   :scale: 70%
   :align: center

   Operator speedup according to the total number of cores used.

.. figure:: ../_static/drum_dem_100M_comp_pourcentage.png
   :scale: 70%
   :align: center

   Operator time ratios at different paralellization scales.


Polyhedra Into A Box
--------------------
