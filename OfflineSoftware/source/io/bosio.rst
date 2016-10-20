
.. _clasio-bosio:

Reading BOS files with Java I/O package
***************************************

The design of CLAS I/O library allows implementation of different 
formats with same interface. And there is a BOS reader implemented
that can read BOS files directly from JAVA. There are also tools
included in COATJAVA package to convert legacy BOS files to EVIO 
format consistent with CLAS12 data format. The advantage of this
is that all standard tools for event manipulation and cuts and corrections
can be used with old data sets.


Preparing to convert
====================

The BOS files created by RECSIS (CLAS-6 event reconstruction program) sometimes
have splitted event buffers and discontinuities of BANKs within an event record.
Java software can deal with some of the fragmentation in the FPACK format, but
not all of them sometimes resulting in mising banks. In order to avoid this issue
it is recomended to pass BOS file through filter which creates clean continues
records out of BOS file. The command to filter BOS is:

.. code-block:: bash

   /group/clas12/packages/bos/bosutility -filter -b "HEADHEVTEVNTECPBCCPBSCPBLCPB" run_28456_A00.bos

The "-b" option specifies which banks should be in the final filtered BOS file, 
if needed other banks can be added as well. Note ! Banks with names shorter than 
4 character have to be completed to 4 characters with " " (space).


Converting BOS to EVIO
======================

The convertor program is called bos2evio and it is included in the coatjava
package. Starting from coatjava 3.0, the bos files are converted into HIPO
files with compression. To convert a bos file on CUE machines use:

.. code-block:: bash

   >/group/clas12/packages/coatjava-3.0/bin/bos2hipo -lz4 myoutput.hipo run018567.A00.B00

The newly created file can be viewed with eviodump program:

.. code-block:: bash

   >/group/clas12/packages/coatjava-3.0/bin/eviodump myoutput.hipo

This will printout banks in each event, to view each bank type the bank name in the prompt.



