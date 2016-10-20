
Running CLAS-12 Reconstruction
******************************

This section describes reconstruction tools for CLAS12 detector.
The reconstruction package is distributed as a tar.gz package and
contains all neccessary files to run reconstruction.

Event Simulation 
================

For sumulating CLAS12 detector the GEMC package is used. The input
to GEMC is an event file in LUND format. The full documentation
on how to run GEMC can be found at: https://gemc.jlab.org.

Since coatjava 3.0 simulated GEMC files have to be first prepared to 
run reconstruction. There is an utility inclused in the coatjava package
to add header information to the file. Run gemc output through:

.. code-block:: bash

   >bin/gemc-evio gemc.evio 10 -1.0 1.0

First argument is the input evio file produced by GEMC.
Second argument is the run number to be written to header of each event (event number is added automatically).
Third and fourth arguments are torus and solenoid magnetic fields respectively.
The output evio file will contain a bank with run configuration. Here is the structure of this bank:

.. code-block:: bash

	 <bank name="RUN" tag="4300" info="Header bank for run condition information">
	    <section name="config" tag="4301"     num="0"   info="Header Describing the Event">
	      <column name="Run"       type="int32"   num="1"   info="Run Number"/>
	      <column name="Event"     type="int32"   num="2"   info="Event Number within the Run"/>
	      <column name="Type"      type="int8"    num="3"   info="Describes the event type (0-for GEMC, 1-for Real Run)"/>
	      <column name="Mode"      type="int8"    num="4"   info="Event Mode (0- normal event with field, 1-cosmic event)"/>
	      <column name="Torus"     type="float32" num="5"   info="Torus Field Scale"/>
	      <column name="Solenoid"  type="float32" num="6"   info="Solenoid Field Scale"/>
	      <column name="RF"        type="float32" num="7"   info="RF reference time"/>
	    </section>
	  </bank>




Running Reconstruction
======================

The software for reconstruction can be downloaded from https://userweb.jlab.org/~gavalian/software/coatjava/
Current version is 3.0 which contains CLARA 4.3 integration and can run multithreaded, Before running reconstruction
program one must run gemc output through utility described in the previous section.
To run reconstruction use:

.. code-block:: bash

   >bin/clara-rec -t 1 -r etc/services/reconstruction.yaml input.evio output.evio

The first argument (-t) indicates how many threads to use, second argument is the configuration
file describing services of reconstruction chain, it is provided with coatjava package. 

The reconstruction can also run without a connection to remote ccdb database. The environmental
variable should be set to use locat database.

.. code-block:: bash

   >setenv CCDB_DATABASE etc/data/database/clas12database.db 

NOTE : Tha path to the database is given with relative path. The code looks for the database in
the directory defined by $CLAS12DIR environmental variable.

