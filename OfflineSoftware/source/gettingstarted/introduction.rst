
Getting Started with Java Software
**********************************

CLAS12 Software is Java based framework based on CLARA for distributed computing. 
It is distributed a single package containing all neccessary files to work with
CLAS12 files (mainly EVIO and HIPO), and to run reconstruction.


Downloading Software
====================

The distribution can be downloaded from web side: https://userweb.jlab.org/~gavalian/software/coatjava

To obtain latest version use:

.. code-block:: bash

   wget https://userweb.jlab.org/~gavalian/software/coatjava/coatjava-4a.0.0.tar.gz

For older versions visit the link above.

Software Documentation
======================

Java-Docs for essential components of the software can be found at:

<http://clasweb.jlab.org/clas12offline/docs/javadoc>

Required Software
=================

CLAS12 software package is written in JAVA and to run the codes and scripts Java>1.7 is required.
To run example scripts groovy package is required. To run on local machines user must install JDK 7
package and groovy.

Running On Jlab CUE
===================

To use latest version of coatjava (CLAS reconstruction package) on CUE environment must be initialized
to use required software do:

.. code-block:: bash

   ifarm> module load java_1.7
   ifarm> use groovy

Starting from version 2.0 of CLAS12 software we switched to JDK 8, in order to use reconstruction package 2.0
user must install JDK 8 on their machine, or on CUE machines load the appropriate module:

.. code-block:: bash

   ifarm> module load java_1.8
   ifarm> use groovy

Then downloaded coatjava package is ready to use. To use up to date development version installed on CUE 
machines include following in command in cshrc file.

.. code-block:: bash

   >source /group/clas12/environment_java.csh

If using version 2.0 of CLAS12 reconstruction on CUE machines a different environment script has to be
sourced:

.. code-block:: bash

   >source /group/clas12/environment_java_8.csh

Running on Local Machines
=========================

To run CLAS12 software on local machine, user is required to have JDK version >= 1.8 and groovy installed. To check your 
JDK version run:

.. code-block:: bash

   >java -version

  java version "1.8.0_51"
  Java(TM) SE Runtime Environment (build 1.8.0_51-b16)
  Java HotSpot(TM) 64-Bit Server VM (build 25.51-b03, mixed mode)

The groovy installation on Mac is very simple, one needs to run:

.. code-block:: bash

  > sudo port install groovy

for port users, or:

.. code-block:: bash

  > brew install groovy

for brew users, for all other systems, check out the instruction on Groovy web site (http://www.groovy-lang.org/download.html):

.. _a link: http://www.groovy-lang.org/download.html

The example script from the web site or from the package must be ran with command line:

.. code-block:: bash

  >$COATJAVA/bin/run-groovy example.groovy

The script sets up the directories and clas paths properly so the scripts can find the packages
of CLAS12 software and all the configuration files that the package needs.



Downloading coatjava
====================

The packaged tarball of complete package with neccessary software and files can be downloaded from:

.. code-block:: bash

   wget https://userweb.jlab.org/~gavalian/software/coatjava/coatjava-1.0.tar.gz

To download the development version of the package use the link:

.. code-block:: bash

   wget https://userweb.jlab.org/~gavalian/software/coatjava/coatjava-2.0.tar.gz

To download newest branch of development use command:

.. code-block:: bash

   wget https://userweb.jlab.org/~gavalian/software/coatjava/coatjava-3.0.tar.gz


Development environment
=======================

To start developing for CLAS12 framework refer to section Development of Code. The library needed
to start development is located on clas maven repository:

.. code-block:: bash
   
   http://clasweb.jlab.org/clas12maven/org/jlab/coat/coat-libs/1.0-SNAPSHOT/

Packages in CoatJava
====================

Here is the list of packages that coatjava distribution contains with links to their javadoc page:

  +-----------------+-----------------------------------------+-------------------------------------------------------------------------------------+
  | Package         |   Description                           |                  Documentation Link                                                 |
  +=================+=========================================+=====================================================================================+
  |  clas-geometry  |  CLAS geometry Package                  |   <http://clasweb.jlab.org/clas12offline/docs/javadocs/clas-geometry/>              |
  +-----------------+-----------------------------------------+-------------------------------------------------------------------------------------+
  |  clas-tools     |  CLAS TOOLS Package                     |   <http://clasweb.jlab.org/clas12offline/docs/javadocs/clas-tools/>                 |
  +-----------------+-----------------------------------------+-------------------------------------------------------------------------------------+
  |  clas-io        |  CLAS EVIO I/O Package                  |   <http://clasweb.jlab.org/clas12offline/docs/javadocs/clas-io/>                    |
  +-----------------+-----------------------------------------+-------------------------------------------------------------------------------------+
  | clas-physics    | Physics Toolkit Library                 |   <http://clasweb.jlab.org/clas12offline/docs/javadocs/clas-physics/>               |
  +-----------------+-----------------------------------------+-------------------------------------------------------------------------------------+
  | jroot           | Java Plotting Library                   |   <http://clasweb.jlab.org/clas12offline/docs/javadocs/jroot/>                      |
  +-----------------+-----------------------------------------+-------------------------------------------------------------------------------------+


