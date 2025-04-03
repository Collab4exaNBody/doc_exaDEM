External Packages
=================

RSA_MPI package
---------------

``RSA MPI`` is a HPC library dedicated to generating random configurations non-intersecting balls, with an unbiased the RSA algorithm. The sequential strategy based on Ebeida et al, 2012. This strategy has been extended to the MPI framework, as decribed in this paper "Parallel and bias-free RSA algorithm for~maximal Poisson-sphere sampling, Josien & Prat (in preparation)".


* Operator name: ``rsa_rnd_rad``
* Parameters:

  * *enlarge_bounds*: Add an empty layer around your domain. Default is 0.0
  * *periodicity*: if set, overrides domain's periodicity stored in file with this value 
  * *bounds*: Override domain's bounds, filtering out particle outside of overriden bound
  * *type* : Define particle, default is 0
  * *expandable* : if set, override domain expandability stored in file
  * *r_min* : Value of the smallest radius possible
  * *r_max* : Value of the largest radius possible
  * *n_max* : Maximal number of particles. Default is 1e16
  * *region* : The region where the sample is generated


YAML examples:

.. code:: yaml

  - rsa_rnd_rad:
     bounds: [[ -4.97 , -4.97 , 1 ] , [ 4.97 , 4.97 , 150 ] ]
     r_min: 0.04
     r_max: 0.05
     n_max: 6000000
     region: MYSUPERZONE
     type: 0
