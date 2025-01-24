I/O and Analysis
================

In this section, we will describe the various operators used for analysis and I/O in DEM simulations.


Configuration
-------------

IO Configuration
^^^^^^^^^^^^^^^^

This operator is used to define the tree structure of output files. By default, exaDEM proposes a tree structure with names defined in the `config_exaDEM` file. You can override this operator yourself by redefining it in the `io_tree` operator.

- Name: `io_config`
- Description: This operator defines the tree structure of output files.
- Parameters:
   * `avg_stress_tensor_name` : Write an output file containing stress tensors.
   * `dir_name` : Main output directory.
   * `interaction_basename` : Write an output file containing interactions.
   * `log_name` : Write an Output file containing log lines.

YAML example:

.. code-block:: yaml

  - io_config:
     dir_name: "ExaDEMOutputDir"
     log_name: "log.txt"
     avg_stress_tensor_name: "AvgStressTensor.txt"
     interaction_basename: "InteractionOutputDir-"

Checkpoint/Restart Operators
----------------------------

Reader Of xyz File
^^^^^^^^^^^^^^^^^^

- Name: `read_xyz`
- Description: This operator reads a file written according to the xyz format.
- Parameters:
   * `bounds_mode` : default mode corresponds to ReadBoundsSelectionMode.
   * `enlarge_bounds` : Define a layer around the volume size in the xyz file. Default size is 0.
   * `file` : File name, this parameter is required.
   * `pbc_adjust_xform` : Adjust the form.

YAML example: 

.. code-block:: yaml

 - read_xyz:
   file: input_file_rigid_surface.xyz
   bounds_mode: FILE
   enlarge_bounds: 1.0 m


How to build your xyz input file of `nb` particles.

.. code-block:: python

 nb
 box_size_x box_size_y box_size_z
 type_0 pos_x_0 pos_y_0 pos_z_0
 type_1 pos_x_1 pos_y_1 pos_z_1
 type_2 pos_x_2 pos_y_2 pos_z_2
 ...
 type_nb-1 pos_x_nb-1 pos_y_nb-1 pos_z_nb-1

This is an example of two particles the same type in a domain [[0,0,0],[10,10,10]].

.. code-block:: python

 2
 10 10 10
 0 2.5 5.0 5.0
 0 7.5 5.0 5.0

Reader Of MPIIO File
^^^^^^^^^^^^^^^^^^^^

- Name: `read_dump_particles`
- Description: This operator reads a dump file with all particle information required to restart the simulation. See operator: @write_dump_particles
- Parameters:
   * `enable_friction` : Enable to write friction information. By default this option is activated.
   * `filename` : Dump file name to read.
   * `bounds` : If set, override the domain's bounds, filtering out particles outside of overriden bounds (AABB = [[infx, infy, infz],[supx, supy, supz]]).
   * `expandable` : If set, override domain expandability stored in file
   * `periodicity` : If set, overrides the domain's periodicity stored in a file with this value (ex: [false,true,false]).
   * `scale_cell_size` : If set, this option rescales cell size. Due to friction storage per cell, ou can only multiply this size by an integer (1,2,4, 8, ...) or divide it by a power of 1/(2^n) (0.5,0.25 ...).
   * `shrink_to_fit` : If set to true and bounds were specified, try to reduce the domain's grid size to the minimum size enclosing fixed bounds.

YAML example:

.. code-block:: yaml

  - read_dump_particles:
      filename: last.dump

.. note::
  This operator is used for spheres and not polyhedra because we need a special reader to read current interaction values containing the friction and moment. Show `read_dump_particle_interaction`.


Restart file script
^^^^^^^^^^^^^^^^^^^

In the `scripts` folder, you have the option of using a script ``restart_template.py`` that will enable you to quickly write the restart section of your simulation by directly retrieving the path to any saved files. This script directly includes whether the simulation mode is spherical or polyhedral. If the mode is polyhedral, the shape file will also be loaded automatically. The script will also check for a file containing driver data and include it. Finally, it will also include the last dump file (highest iteration). 

Option:

* `directory`: Output file path, the default path is `ExaDEMOutputDir`. Note that it should contains a subdirectory named: `CheckpointFiles` and this output file path is defined into the operator ``io_tree``.

Example: 

.. code-block:: bash

   python3 ~/exaDEM/scripts/restart_template.py --directory SpheresMovableWallDir

Output:

.. code-block:: bash

   Restart directory: SpheresMovableWallDir
   Particle mode: Spheres 
   Last iteration identified: 29000
   Here s a template for restarting your simulation at the last saved iteration: 
   
   includes:
     - config_spheres.msp
     - SpheresMovableWallDir/CheckpointFiles/driver_0000029000.msp
   
   input_data:
     - read_dump_particle_interaction:
        filename: SpheresMovableWallDir/CheckpointFiles/exadem_0000029000.dump

Read Shape File
^^^^^^^^^^^^^^^

The purpose of this operator is to add shapes to a collection of shapes. This operator can be called as many times as desired. However, if you add the same shape multiple times, it will create duplicates. Additionally, the shapes will be ordered according to the order of reading, meaning that type 0 will be associated with the first shape from the first input file. Furthermore, this operator will automatically create a polydata for each shape, which will be used for displaying the polyhedra using ParaView.

.. note::

  The output Paraview file does not incorporate the "spherical" characteristics of polyhedra, i.e. surfaces are created by connecting the centers of vertices, edges are straight lines (instead of cylinders), and vertices are points (instead of spheres).

* `read_shape_file` :
   * `filename`: Input file name, no default name.

Warnings:

.. warning::

  * This operator takes on ASCII files.
  * This operator is not typo-proof and will ignore problematic values.
  * Do not define a shape "driven" such as a wall or a cylinder because the cell diameters and the cutoff radius for creating interaction lists are derived from the shapes of the polyhedra. These should be defined in specific operators if they have an analytical shape. If they have particular shapes with many facets, please use the STL mesh reader.

YAML example:

.. code-block:: yaml

  - read_shape_file:
     filename: shapes.shp

Example of a shape:

.. code-block:: python

  <
  name Octahedron
  radius 0.1
  preCompDone y
  nv 6
  0.2310789034541148 -0.2310789034541148 0.0
  0.2310789034541148 0.2310789034541148 0.0
  0.0 0.0 0.32679491924311227
  -0.2310789034541148 -0.2310789034541148 0.0
  -0.2310789034541148 0.2310789034541148 0.0
  0.0 0.0 -0.32679491924311227
  ne 12
  0 1
  2 1
  2 0
  0 3
  2 3
  3 4
  4 2
  4 1
  5 0
  5 1
  5 4
  5 3
  nf 8
  3 0 1 2 
  3 2 3 4 
  3 1 2 4 
  3 0 2 3 
  3 0 5 1 
  3 0 5 3 
  3 3 5 4 
  3 4 5 1 
  obb.extent 0.33107890345411484 0.33107890345411484 0.4267949192431123
  obb.e1 1.0 0.0 0.0
  obb.e2 0.0 1.0 0.0
  obb.e3 0.0 0.0 1.0
  obb.center 0.0 0.0 0.0
  position 0.0 0.0 0.0
  orientation 1.0 0.0 0.0 0.0
  volume 0.16666666666666666
  I/m 0.04999999999999999 0.04999999999999999 0.04999999999999999
  >

Example of `Octahedron.vtk` with paraview:

.. image:: ../_static/octahedron.png
   :width: 300pt
   :align: center

Writer Of MPIIO Files
^^^^^^^^^^^^^^^^^^^^^

- Name: `write_dump_particles`
- Description: This operator writes a dump file with all particle information required to restart the simulation. See operator: @read_dump_particles.
- Parameters:
   * `compression_level` Zlib compression level.
   * `filename` Dump output file name.
- Default behaviour: the default name is defined by : `- timestep_file: "exaDEM_%09d.dump` and piloted by `simulation_dump_frequency: 1` in the operator `global`.

.. note::
  This operator is defined in the default `ExaDEM` operator named `dump_data_particles`. 

Writer Of XYZ Files
^^^^^^^^^^^^^^^^^^^

- Name: `write_xyz_generic`
- Description: This operator writes a txt file (`.xyz`) with all specified fields.
- Parameters:
  * `fields`: array of fieldsets. Example: ``[ id, velocity, radius ]``
  * `filename`: name of the output file.
  * `units`: array of units. Example: ``{ velocity: "m/s", radius: "m" }``

.. note:: 
  The first line of the output file contains the number of particles. The second line contains the “lattice” description, useful when using ovito.

YAML example: Replaces MPIIO output files with xyz files. 

.. code-block:: yaml

  dump_data_xyz:
    - timestep_file: "dem_pos_vel_%09d.xyz"
    - write_xyz_generic:
       fields: [ id, velocity, radius ]
       units: { velocity: "m/s", radius: "m" }

  iteration_dump_writer:
    - dump_data_xyz

  global:
    simulation_dump_frequency: 500


To process these files, a sample script is provided in ``scripts/post_processing/profile_pos_vel.py``. This is a minimal, easily modifiable post-processing file that calculates the averages of all position and velocity components.

Output file: [mean_r_v.pdf]

.. image:: ../_static/Analyses/mean_r_v.png
   :width: 500pt
   :align: center



Dump Paraview For Polyhedra
^^^^^^^^^^^^^^^^^^^^^^^^^^^

In exaDEM, there are two ways to display polyhedra with Paraview: 
   * The first is to directly display the vertices of the polyhedra and the surfaces in parallel VTP (PolyData). However, no fields associated with the polyhedra are available, such as velocity or density.
   * The second solution is to use the generic Paraview output of `exaDEM` by adding the orientation field and the homethety field (optional). Then, it's possible to associate a mesh with each point, such as an `octahedron.vtk` file generated by `read_shape_file`, to each point by associating it with a size (`field::homothety`) and a quaternion (`field::orient`).

.. note::
  Only the default behavior when the `config_polyhedra.msp file` is used is the option 2 that offers more possibilities. In addition, it is important to note that paraview does not include the layer of shape->radius size, i.e. faces are displayed according to the vertex centers.


* Option 1: `write_paraview_generic`
   * `binary_mode` [BOOL] : paraview format file, default is true.
   * `compression` [STRING] : level of compression, default is "default" for vtkZLibDataCompressor.
   * `filename` [STRING]: basename of the parallel paraview output files, default is "output". 
   * `write_ghost` [BOOL]: dump ghost particles, default is false.
   * `write_box` [BOOL]: write box information in a box.vtp file, default is true. 
   * `write_external_box` [BOOL]: write external box (ghost area), default is false. 
   * This operator is based on this function: `ParaviewWriteTools::write_particles`.

YAML example:

.. code-block:: yaml

  write_paraview_generic:
    binary: false
    write_ghost: false
    fields: ["vx","vy","vz","id","orient"]

How to use it with Paraview:

- Firstly, we need to load our reference mesh (Octahedron.vtk) in our case.
- Secondly, load your particle file into the Paraview folder (default name in exaDEM for the paraview_generic operator).
- Thirdly, choose the 3D Glyphs representation and the coloring.

.. image:: ../_static/tuto1_dump_polyhedra.png
   :width: 250pt
   :align: center

- Fourthly, in the Glyph Parameters section, choose "Orient" with the orientation mode "Quaternion" and as orientation vectors: "orient". To change the size, you can check Scaling and add the Scale Array you wish. Finally, in the Glyph Type dropdown menu, select "Pipeline Connection" and in Input, choose "Octahedron.vtk".

.. image:: ../_static/tuto2_dump_polyhedra.png
   :width: 250pt
   :align: center

Result for a simulation of 1000 Octahedra falling in a cylinder colored by their ID:

.. image:: ../_static/tuto3_dump_polyhedra.png
   :width: 500pt
   :align: center

* Option 2: write_paraview_polyhedra
   * `filename` : Name of paraview file, there is no default name. Note that in `ExaDEM`, filename is defined in the default execution stream.
   * `mpi_rank` : Add a field containing the mpi rank. [optional]

YAML example:

.. code-block:: yaml

  particle_write_paraview_generic:
    - write_paraview_polyhedra:
       mpi_rank: true


Example with 850,000 octahedra:

.. image:: ../_static/850kpolyzoom.png
   :width: 500pt
   :align: center

.. note::
	This operator is rather limited in terms of visualization, so we now advise you to use option 1, which offers more possibilities (field display) and less memory-intensive files. 


Analysis
--------

In this section, we will describe the operators related to the usage of polyhedra or spheres.

Dump Paraview With OBBs
^^^^^^^^^^^^^^^^^^^^^^^

This operator allows you to display OBBs around polyhedra in paraview. These files are stored in different files from those used to store polyhedron information. By default, these files are available in the directory `ExaDEMOutputDir/ParaviewOutputFiles/` under the format `obb_%010d.pvtp`. The fields associated with OBBs are the polyhedron ID and type.

* `write_paraview_obb_particle`:
   * `basedir` : Name of the directory where paraview files will be written
   * `basename` : Name of paraview file, there is no default name. Default is "obb".
   * `timestep` : Current simulation time is defined.

.. note::
  This operator is called after `write_paraview_generic` and is triggered by `simulation_paraview_frequency` called into the global operator.

.. warning::
  This operator doesn't work for simulations with spheres.

YAML example:

.. code-block:: yaml

  - write_paraview_obb_particle

Output example:

.. image:: ../_static/obb_cylinder_start.png
   :width: 350pt
   :align: center

.. image:: ../_static/obb_cylinder_end.png
   :width: 350pt
   :align: center


Dump Contact Network
^^^^^^^^^^^^^^^^^^^^

This operator is used to visualize the contact network between polyhedra using ParaView. For each active contact/interaction, we assign the value of the normal force calculated in Contact's law. You can enable this option, which will be automatically triggered at the same time as the other paraview files, with the option ``enable_contact_network: true`` in global. See examples: "Polyhedra/Example 2: Octahedra in a Rotating Drum" and "Spheres/Example 1: Rotating drum".

* `dump_contact_network`:
   * `filename` : Name of the paraview file, there is no default name.  
   * `timestep` : Current simulation time is defined.

YAML example:

.. code-block:: yaml

  - timestep_paraview_file: "ParaviewOutputFiles/contact_network_%010d"
  - dump_contact_network

.. code-block:: yaml

  global:
    enable_contact_network: true


Here is an example of 216 polyhedra after a fall into a cylinder, left the simulation and right the contact network:

.. image:: ../_static/contact_network_example.png
   :width: 500pt
   :align: center


Comments / Extensions:

* This operator can be modified to display more values per contact. To achieve this, you need to change the type of `StorageType` in the `NetworkFunctor` structure. Then, you'll need to populate this function in the operator `() (exaDEM::Interaction* I, const size_t offset, const size_t size)`. Finally, you'll need to add a field in write_pvtp and include this field in `write_vtp`.
* Currently, this operator doesn't take particularly long to execute and isn't called frequently. However, it doesn't benefit from any shared-memory parallelization (OpenMP) because the network storage is implemented using a `std::map`. 

Dump Interaction Data
^^^^^^^^^^^^^^^^^^^^^

This feature outputs the main information for each interaction. This feature has been implemented to enable post-simulation analysis.  
An option has been added to the contact_polyhedron and contact_sphere operators to output interaction data as a CSV file. To activate it, simply modify the value of ``analysis_interaction_dump_frequency`` in the operator block ``global``. 

Output files are located in the `ExaDEMOutputDir/ExaDEMAnalysis` folder. For each iteration (XXX) with file writing, a folder containing an interaction file is created, such as:  `Interaction_XXX/Interaction_XXX_MPIRANK.txt`.

For each interaction, we write:

- The particle identifier i [uint64_t],
- The particle identifier j [uint64_t],
- The sub-identifier of the particle i [int], 
- The sub-identifier of the particle j [int],
- The interaction type [int <= 13],
- The deflection / overlap [double <= 0],
- The contact position [Vec3d], 
- The normal force [Vec3d], 
- The tangential force [Vec3d].


.. warning::

  Inactive interactions have been filtered out when writing output files. In addition, symmetrized interactions are stored one time. 

.. note::

  An example is available in: example/polyhedra/analyses/interaction.msp 


``ExaDEM`` also offers post-processing scripts for basic interaction analyses. The scripts can be used as a basis for developing other analyses according to need. The first available script is `interaction_summary.py` : 

- Read all interaction files
- Plot the number of interactions per type as a	 function of the timestep (`types.pdf`)
- Plot the number of interactions as a function of the timestep (`count.pdf`)

How to run this script:

.. code-block:: bash

  cd ExaDEMOutputDir/ExaDEMAnalyses
  python3 PATH_TO_ExaDEM/scripts/post_processing/interaction_summary.py

Output file examples:

Simulation: near 104,000 octahedral particles over 200,000 timesteps of 5.10^{-5} s falling into a cylinder.


.. image:: ../_static/Analyses/analyses.png
   :width: 500pt
   :align: center

- types.pdf

.. image:: ../_static/Analyses/types.png
   :width: 500pt
   :align: center

.. note::

  Symmetrized interactions are counted twice within the ``interaction_summary.py`` python script.

- count.pdf

.. image:: ../_static/Analyses/count.png
   :width: 500pt
   :align: center

Interaction Summary
^^^^^^^^^^^^^^^^^^^^

This operator allows displaying the total number of interactions, both total and active. An interaction is considered active if there is contact ands consequently, if the cumulative friction is different from Vec3d{0,0,0}. It also enables the separation of different types of interactions: Vertex-Vertex, Vertex-Edge, Vertex-Face, and Edge-Edge.

- Name: `stats_interactions`
- No parameter.
- Tip: Add this operator when performing recurring but infrequent operations such as Paraview outputs or checkpoint output files (see YAML example). 

YAML example:

.. code-block:: yaml

   +dump_data_paraview:
     - stats_interactions

Output example:

.. code-block:: bash

  ==================================
  * Type of interaction    : active / total 
  * Number of interactions : 42058 / 41943
  * Vertex - Vertex        : 0 / 0
  * Vertex - Edge          : 625 / 625
  * Vertex - Face          : 5546 / 5546
  * Edge   - Edge          : 31698 / 31698
  * Vertex - Cylinder      : 0 / 0
  * Vertex - Surface       : 0 / 0
  * Vertex - Ball          : 0 / 0
  * Vertex - Vertex (STL)  : 0 / 0
  * Vertex - Edge (STL)    : 0 / 0
  * Vertex - Face (STL)    : 4060 / 4074
  * Edge   - Edge (STL)    : 0 / 0
  * Edge (STL) - Vertex    : 0 / 0 
  * Face (STL) - Vertex    : 0 / 0
  ==================================

Global Stress Tensor
^^^^^^^^^^^^^^^^^^^^

A stress tensor for a given particle is computed such as: 

.. math::

  \sigma_{loc}=\sum_{ij \in I}[f_{ij} c_{ij}^T]

With ``I`` the active interactions, :math:`f_{ij} = (fx_{ij},fy_{ij},fz_{ij})` the forces between the particle `i` and `j`, and :math:`c_{ij} = r_i - p_{ij}` the vector between the center of the particle `i` noted :math:`r_i` and the contact position of the interaction ``I`` named :math:`p_{ij}`. 

And the total stress of the system : :math:`\sigma =  \frac{1}{V} \sum_{loc} [\sigma_{loc}]`, with ``V`` the volume.


Stress tensor calculation is performed by the ``stress_tensor`` operator, and writing to a .txt output file is performed by the ``write_stres_tensor`` operator. To trigger the writing of the stress tensor, simply declare the ``analysis_dump_stress_tensor_frequency`` variable to the frequency chosen in the global operator of your YAML file (`.msp`), which by default is set to -1.

YAML example:

.. code-block:: yaml

  global:
    simulation_end_iteration: 150000
    simulation_log_frequency: 1000
    simulation_paraview_frequency: 10000
    analysis_dump_stress_tensor_frequency: 1000

**For further information**

This frequency triggers several things. When passing through the ``Contact Force`` operator, the list of interactions / normal force / tangential force iq stored in the classifier. The stress tensor is then calculated in ``global_stress_tensor`` and written in ``write_stress_tensor``. By default, volume is calculated from the simulation volume using the ``compute_volume`` operator. So, by default, the frequency will trigger the chaining of these three operators: 

.. code-block:: yaml

  compute_volume:
    - domain_volume
  
  global_stress_tensor:
    - average_stress_tensor
  
  dump_stress_tensor_if_triggered:
    condition: trigger_write_stress_tensor
    body:
      - compute_volume
      - global_stress_tensor
      - write_stress_tensor

In the case of a particle deposit or other simulation where the simulation domain does not correspond to the simulation volume, you can either implement your ``my_volume`` operator and replace the ``compute_volume`` operator block such as:

.. code-block:: yaml

  compute_volume:
    - my_volume:
       my_param1: x
       my_param2: y
       my_param3: z

If you want to directly assign the value of a fixed-size volume, we advise you to add these lines to your input file: 

.. code-block:: yaml

	compute_volume: nop

	global_stress_tensor:
		- average_stress_tensor:
			 volume: 21952

A usage example is available at the following address: `example/polyhedra/analyses/write_avg_stress.msp`. It involves dropping a set of hexapods into a box and watching the stress tensor evolve over time.

.. |anastart| image:: ../_static/avgStressStart.png
   :width: 250pt

.. |anaend| image:: ../_static/avgStressEnd.png
   :width: 250pt

|anastart| |anaend|


A gnuplot script is available at `scripts/post_processing/avg_stress.gnu` to quickly plot lines:

.. code-block:: bash

	set key autotitle columnhead
	N = system("awk 'NR==1{print NF}' AvgStressTensor.txt")
	plot for [i=2:N] "AvgStressTensor.txt" u 1:i w l
	set key autotitle columnhead
	set term png
	set output "avgStress.png"
	replot


.. image:: ../_static/Analyses/avgStress.png
   :width: 400pt
   :align: center


Particle Counter
^^^^^^^^^^^^^^^^

The purpose of this operator is to count the number of particles per type in a particular region. 

Operator:

* Name: ``particle_counter``
* Parameters:

  * `name`: Filename. Default is: ParticleCounter.txt,
  * `types`: List of particle types (required, [0,1,2, ...]),
  * `region`: Choose the region, default is the domain simulation.

To use this operator, the simplest way is to define the analysis frequency (all) in the global operator (``simulation_analyses_frequency``) and add the ``particle_count`` operator to the operator ``analyses``, as in the following example (see ``example/polyhedron/analysis/particle_counter.msp``: 	

.. code-block:: yaml

  global:
    simulation_analyses_frequency: 10000

  analyses:
    - particle_counter:
       name: "ParticleTypes0And1.txt"
       types: [0,1]
       region: BOX


One possibility for post-processing is to use gnuplot with the following commands: 

.. code-block:: bash

  set key autotitle columnheade
  set style data lines
  plot for [i=2:3] 'ExaDEMOutputDir/ExaDEMAnalyses/ParticleTypes0And1.txt' using 1:i smooth mcsplines


.. |countergif| image:: ../_static/particle_counter.gif
   :width: 300pt

.. |counterplot| image:: ../_static/particle_counter_plot.png
   :width: 300pt


Results:

+--------------------------+--------------------------+
| .. centered:: Particle Counter Output               |
+--------------------------+--------------------------+
| .. centered:: Simulation | .. centered:: Plot       |
+==========================+==========================+
| |countergif|             | |counterplot|            |
+--------------------------+--------------------------+

Barycenter
^^^^^^^^^^

The purpose of this operator is to count the number of particles per type in a particular region. 

Operator:

* Name: ``particle_barycenter``
* Parameters:

  * `name`: Filename. Default is: ParticleBarycenter.txt,
  * `types`: List of particle types (required, [0,1,2, ...]),
  * `region`: Choose the region, default is the domain simulation.

To use this operator, the simplest way is to define the analysis frequency (all) in the global operator (``simulation_analyses_frequency``) and add the ``particle_count`` operator to the operator ``analyses``, as in the following example (see ``example/polyhedron/analysis/particle_barycenter.msp``: 	

.. code-block:: yaml

  global:
    simulation_analyses_frequency: 10000

  analyses:
    - particle_barycenter:
       name: BaraycenterBox.txt
       types: [0,1]
       region: BOX
    - particle_barycenter:
       name: Baraycenter.txt
       types: [0,1]

One possibility for post-processing is to use gnuplot with the following commands: 

.. code-block:: bash

  set key autotitle columnheade
  set style data lines
  set title font "Barycenter per particle type" 
  set xlabel "Position X" 
  set ylabel "Position Z"
  plot "PolyhedraAnalysisBarycenterDir/ExaDEMAnalyses/Baraycenter.txt" u 2:4 w l title "Poly"
  replot "PolyhedraAnalysisBarycenterDir/ExaDEMAnalyses/Baraycenter.txt" u 5:7 w l title "Octahedron"
  set terminal png
  set output "barycenter_plot.png"
  replot

.. |barycentergif| image:: ../_static/particle_barycenter.gif
   :width: 300pt

.. |barycenterplot| image:: ../_static/barycenter_plot.png
   :width: 300pt

+--------------------------+--------------------------+
| .. centered:: Particle Counter Output               |
+--------------------------+--------------------------+
| .. centered:: Simulation | .. centered:: Plot       |
+==========================+==========================+
| |barycentergif|          | |barycenterplot|         |
+--------------------------+--------------------------+
  
