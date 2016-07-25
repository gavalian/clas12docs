
.. _hipo:

***********
HIPO Files 
***********

As part of CLAS12 software framework HIPO library was developed for storing 
reconstruction output and possibly the final DSTs. HIPO provides random access
to compressed data sets and has no limitation on file size. It is useful for
chaining many evio files together to save on storage and have ability to
process them at once.

Creating a Hipo File
====================

There is a Hipo convertor provided with coatjava distribution. To combine
multiple EVIO files into one HIPO file use the command.

.. code-block:: java

  >bin/hipo-writer -lz4 -b ALL output.hipo input1.evio input2.evio input3.evio

This will create a large file, reduced in size due to internal compression.
The events stored inside are EVIO events, they are grouped together and indexed
for easy and fast random access. The lz4 flag tells converter to compress the output
with LZ4 compression, other options are "-u" for uncompressed and "-gzip" for GZIP
compression. The second flag is for controlling the output banks. The flag -b ALL
indicates that entire event will be written to the output, without any alteration.
With "-b" option also list of banks can be specified to be in the output. For example:

.. code-block:: java

  >bin/hipo-writer -lz4 -b PCAL:EC output.hipo input1.evio input2.evio input3.evio

will write only banks for PCAL and EC in the output. This includes all the section 
that are declared in the XML file as part of PCAL and EC, namely "PCAL::true",
"PCAL::dgtz", "EC::true" and "EC::dgtz". Entire Reconstruction bank structures can be
also written to output, for example:

.. code-block:: java

  >bin/hipo-writer -lz4 -b DC:HitBasedTrkg output.hipo input1.evio input2.evio input3.evio

will produce output with all DC banks and Hit based tracking banks.
To produce DST's which contain output from Event Builder alone, one has to use:

.. code-block:: java

  >bin/hipo-writer -lz4 -b EVENTHB:GenPart output.hipo input1.evio input2.evio input3.evio

This will output reconstructed particle bank, associated detector response bank and generated
particle banks.

Reading Hipo Files
==================

Reading Hipo files is not different from reading EVIO files. The DataSource
objects are interfaced inside of our framework so they all behave in exactly 
same way.

.. code-block:: java

  import org.jlab.evio.clas12.*
  import org.jlab.data.utils.DictionaryLoader;
  import org.jlab.clas.tools.utils.*;
  import org.jlab.hipo.*;

  filename = args[0];

  HipoDataSource reader = new HipoDataSource();
  reader.open(filename);
  int nevents = reader.getSize();

  int counter = 0;
  for(int i = 0; i < nevents; i++){
    EvioDataEvent  event = (EvioDataEvent) reader.gotoEvent(i);
    event.show();
    counter++;
  }
  System.out.println("  procecessed " + counter + "  events");

