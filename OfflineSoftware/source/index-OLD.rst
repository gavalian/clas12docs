.. Clas12 Docs documentation master file, created by
   sphinx-quickstart on Thu Mar 27 11:11:56 2014.
   You can adapt this file completely to your liking, but it should at least
   contain the root `toctree` directive.

CLAS12 Offline software documentation
=====================================

Welcome to CLAS12 offline software documentation. This section contains topical 
documentation on offline common tools used for building CLAS12 reconstruction
and calibration software. 

Standard Java Library For CLAS12:
---------------------------------

.. toctree::
   :maxdepth: 1

   gettingstarted/introduction


CLAS12 Simulation/Reconstruction
---------------------------------

* `Running gemc at JLab <https://gemc.jlab.org/gemc/html/documentation/runningJLab.html>`_

.. toctree::
   :maxdepth: 1

   reconstruction/introduction
   analysis/introduction
   reconstruction/fastMonteCarlo

CLAS12 Online

.. toctree::
   :maxdepth: 1

   online/rawdataonline.rst   

CLAS12 Input/Output Packages:
-----------------------------

.. toctree::
   :maxdepth: 1

   io/readingRawEvioFiles
   io/clasio
   io/bosio
   io/hipo

Plotting Package:
-----------------

.. toctree::
   :maxdepth: 1

   plotting/graph_plotting
   plotting/histogram_plotting
   plotting/directories_and_trees

Geometry Package:
-----------------

.. toctree::
   :maxdepth: 1

   geometry/geometryprimitives
   geometry/geometrycontainers

CCDB Database:
--------------

.. toctree::
   :maxdepth: 1

   database/readingCalibrationConstants

Code Development:
-----------------

.. toctree::
   :maxdepth: 1

   calibration/monitoringPlugins
   development/introduction
   calibration/calibrationModule

..   reconstruction/introduction
..   analysis/introduction

..   io/clasio.rst
..   io/bosio
..   io/evioraw


..   rootio/introduction
..   io/introduction
..   services/introduction
..   database/introduction
..   calibration/introduction      


Indices and tables
==================

* :ref:`genindex`
* :ref:`modindex`
* :ref:`search`
