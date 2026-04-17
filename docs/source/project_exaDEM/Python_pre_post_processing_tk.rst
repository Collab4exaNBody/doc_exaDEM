Python pre/post processing toolkit
=========

The python scripts files can be found in the ``scripts`` folder.

Overview
-------------------
The exaDEM/script folder provides a collection of Python scripts dedicated to pre-processing and post-processing tasks for exaDEM simulations.
Its goal is to streamline the preparation of input data and the analysis of simulation outputs.

The project is organized into two main components. 
- a core Python library (``scripts/lib``) gathers all generic functionalities, including data structures (particles, interactions, geometries), input/output utilities, and various computational tools.
- standalone scripts (located in ``scripts/pre_processing`` and ``scripts/post_processing``) implement specific use cases such as configuration generation or result analysis. These scripts rely on the shared library..

Environment :
The command (``source /scripts/setenv.sh``) automatically configures the ``PYTHONPATH`` so that the internal library can be used without requiring installation. 
Once the environment is sourced, users can run the scripts from any location while still accessing the project’s modules.

Pre-processing
-----------------

The pre-processing tools are responsible for generating simulation-ready input files from geometric or mesh-based representations.

tess2rockable.py
~~~~~~~~~~~~~~~~

Overview
^^^^^^^^
This script converts a **Neper .tess tessellation file** into a complete **Rockable-compatible input configuration**.  
It performs:

- parsing of polyhedral cells
- geometric reconstruction of Voronoi-like structures
- Minkowski erosion to account for particle radius
- generation of Rockable shapefile (.shp)
- generation of a full simulation configuration with “sticked particles”

The goal is to transform a geometric tessellation into a physically usable DEM input.

Workflow
^^^^^^^^
1. Read Neper tessellation file (`.tess`)
2. Extract vertices, faces, edges, and polyhedra
3. Compute local face normals oriented toward cell centers
4. Perform Minkowski erosion of each cell:
   - each face plane is shifted by a distance `gap`
   - intersections of shifted planes define the eroded polyhedron
5. Compute:
   - volume
   - center of mass
   - inertia tensor (Monte Carlo integration)
6. Write:
   - Rockable shapefile (`.shp`)
   - Rockable configuration file (`_sticked.conf`)

Inputs
^^^^^^^^
- ``input.tess`` (Neper tessellation file)
- ``radius`` (float)
  - particle radius used for erosion and stick distance

Outputs
^^^^^^^^
- ``output.shp``
  - Rockable-compatible geometry file
  - contains vertices, faces, edges, inertia tensor, volume

- ``output_sticked.conf``
  - full simulation configuration
  - includes:
    - particle initialization
    - interaction laws
    - gravity and timestep settings
    - stickVerticesInClusters parameter

Command line usage
^^^^^^^^
.. code-block:: bash

   python tess2rockable.py input.tess 0.1 output.shp

Physical assumptions
^^^^^^^^
- Cells are assumed to behave as rigid polyhedra
- Erosion ensures no overlap between particles
- Contact behavior is governed by Rockable's ``StickedLinks`` law
- Stick distance is set to ``4 × gap`` (where gap = 2 × radius)


Post-processing
-------------------------------------

The post-processing tools are used to analyze DEM simulation outputs and extract physical quantities such as stress tensors, strains, and boundary forces.

compute_clusters_stresses_tensors.py
~~~~~~~~

Overview
^^^^^^^^

This script computes:
- stress tensors per cluster of particles
- global wall forces
- global deformation (strain) of the particle assembly

It processes time series of:
- ``*.xyz`` particle snapshots
- corresponding interaction files

The goal is to extract continuum-scale mechanical quantities from discrete simulations.

Workflow
^^^^^^^^
For each simulation timestep:

1. Load particle snapshot (`.xyz`)
2. Load corresponding interaction file
3. Build particle index and cluster grouping
4. Compute stress tensor: 
   - contact forces
   - contact positions
   - normalization by cluster volume (or particle density)

5. Compute wall forces:
   - separation of contacts by spatial position
   - aggregation of forces on top/bottom/left/right boundaries

6. Compute deformation:
   - global bounding box of particles
   - reference configuration stored at first timestep
   - strain computed as relative variation

Inputs
^^^^^^^^
- particle files:
  - ``*.xyz`` (DEM snapshots)
  - format: position, velocity, cluster id, etc.
  - generated in ExaDEM datasets by : 
  .. code-block::
     analyses:
     - timestep_file: "OutputRockableTestDir/ExaDEMAnalyses/dem_pos_vel_%09d.xyz"
     - write_xyz:
        fields: [ id, velocity, fx, fy, fz, vrot, arot, cluster, mass ]

- interaction files:
  - ``InteractionOutputDir-*/InteractionOutputDir-*_0.txt``
  - generated in ExaDEM datasets by : 
  .. code-block::
    global:
      symetric: true
      analysis_interaction_dump_frequency: 500

Outputs
^^^^^^^^
- ``clusters_stresses_tensor.txt``

  Time series of stress tensors per cluster:

  .. code-block::

     time cluster_id sxx syy szz sxy sxz syx syz szx szy

- ``wall_forces.txt``

  Time series of global mechanical response:

  .. code-block::

     time delta_x delta_y delta_z eps_x eps_y eps_z Fx Fy Fz

Command line usage
^^^^^^^^
.. code-block:: bash

   python compute_clusters_stresses_tensors.py --dir . --density 2500

Arguments:
- ``--dir`` : directory containing simulation outputs
- ``--density`` : particle material density (used for stress normalization)


Physical meaning
^^^^^^^^
- Stress tensor:
  - computed from contact force moments
  - normalized by cluster volume (or mass/density approximation)

- Wall forces:
  - represent macroscopic boundary reactions
  - split between top/bottom regions

- Strain:
  - computed from initial bounding box
  - gives global deformation of the granular assembly

Limitations
^^^^^^^^
- Assumes correct pairing between `.xyz` and interaction files




