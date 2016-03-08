
.. _clasio-raw:

Reading Raw data from EVIO
**************************

The clas-io library provides utilities to read/write evio files consistent with CLAS12
conventions. It has classes to manipulate RAW (DAQ) EVIO files as well as GEMC output
EVIO files with dictionary based bank structures.

Introduction
============

There are several modes used in CLAS12 to store raw data. The raw data is stored in composite 
bank structures, where the tag number of the bank represents the CRATE the data is collected 
from then inside there is composite data structure to contain information about SLOT and CHANNEL
followed by the data (typically different for each MODE). MODE=1 refers to raw mode for Flash ADC
pulse where the entire PULSE shape is stored for each CHANNEL. In MODE=3 integrated PULSE information
is stored. 


Reading Raw (DAQ) Files
=======================

Following are examples of groovy scripts used to read evio RAW data files. The examples 
can be found in coatjava package in directory sctipts/evio.
To read raw files one must use:

.. code-block:: java

    import org.jlab.evio.clas12.*;
    import org.jlab.clas12.raw.*;
    import org.jlab.evio.decode.*;
    import org.jlab.clas.detector.*;

    outputFile = args[0];

    EvioSource  reader = new EvioSource();
    reader.open(outputFile);

    EvioEventDecoder decoder = new EvioEventDecoder();

    int counter = 0;

    while(reader.hasEvent()){

       EvioDataEvent event = reader.getNextEvent();
       decoder.list(event);
       counter++;
    }

    reader.close();


This script will read each event and print out branches that correspond to crate numbers.
The printout looks like:

.. code-block:: bash

   [EVIO DATA EVENT LIST]
   EVIO Branch tag = 4 (4) num = d7 (215)
     evio leaf : tag = e10a (57610) num = 0 (0) length = 4
     evio leaf : tag = e111 (57617) num = 0 (0) length = 35
     evio leaf : tag = e10f (57615) num = 0 (0) length = 5

Where tag corresponds to CRATE number and num is changing from 0 to 255 depending on the event number 
to keep track of event sequence. The leafs have different structure and contain data read from the 
crate. The data is written in composite bank format, and it is the branch with tag=57617 in this example.
The tags can vary depending on the mode used to write data (raw ADC mode or Pulse ADC mode).
The EvioRawDataSource class will automatically recognize the tags and parse them individually depending 
on the data mode.

Raw Data Banks
==============

For each CRATE an array of the hits (data) can be read from the event, the EvioRawDataSource class
automatically detects the MODE of the data (which is based on the TAG number of the leaf) and constructs 
a list of RawDataEntry class which has decoded information from the CRATE.

.. code-block:: java

  ...
  List<DetectorRawData> rawData = decoder.getDataEntries(event,4);

  for(DetectorRawData data : rawData){
      System.out.println(data);
  }
  ...

The code above will produce a different printout depending on which MODE data appears in the bank.
An example printout is from SVT data bank (which is MODE=4).

.. code-block:: bash

  C/S/C [   4    3  2599] S/L/C [   0    0     0]  SVT H/CID/C/T/A [     1      2     39    237      7]
  C/S/C [   4    3  2910] S/L/C [   0    0     0]  SVT H/CID/C/T/A [     1      3     94    238      2]
  C/S/C [   4    5  2419] S/L/C [   0    0     0]  SVT H/CID/C/T/A [     1      1    115    237      4]
  C/S/C [   4    5   804] S/L/C [   0    0     0]  SVT H/CID/C/T/A [     0      3     36    237      2]
  C/S/C [   4    5   805] S/L/C [   0    0     0]  SVT H/CID/C/T/A [     0      3     37    237      7]
  C/S/C [   4    5  2933] S/L/C [   0    0     0]  SVT H/CID/C/T/A [     1      3    117    237      5]
  C/S/C [   4    5  3126] S/L/C [   0    0     0]  SVT H/CID/C/T/A [     1      4     54    237      7]
  C/S/C [   4    5   569] S/L/C [   0    0     0]  SVT H/CID/C/T/A [     0      2     57    237      1]
  C/S/C [   4    5   570] S/L/C [   0    0     0]  SVT H/CID/C/T/A [     0      2     58    237      1]
  C/S/C [   4    5   806] S/L/C [   0    0     0]  SVT H/CID/C/T/A [     0      3     38    238      0]
  C/S/C [   4    5  2365] S/L/C [   0    0     0]  SVT H/CID/C/T/A [     1      1     61    238      1]
  C/S/C [   4   17  2337] S/L/C [   0    0     0]  SVT H/CID/C/T/A [     1      1     33    237      4]
  C/S/C [   4   17  2374] S/L/C [   0    0     0]  SVT H/CID/C/T/A [     1      1     70    237      3]
  C/S/C [   4   17  3114] S/L/C [   0    0     0]  SVT H/CID/C/T/A [     1      4     42    237      5]

In this printout first 3 numbers are CRATE/SLOT/CHANNEL combination. The second set of 3 numbers are 
SECTOR/LAYER/COMPONENT which initally are set to '0' since no translation table was yet applied to the 
list of the hits. Following numbers are dependnent on which MODE the data is in, in this exmaple there 
are HALF/CHIPID/COMPONENT/TDC/ADC.

It is possible to read all entries from all CRATES available in the EVENT, one has to ommit the CRATE parameter
int the call to eader.getDataEntries().

.. code-block:: java

  ...
  List<DetectorRawData> rawData = decoder.getDataEntries(event);

  for(DetectorRawData data : rawData){
      System.out.println(data);
  }
  ...

Working with translation tables
===============================

There is a standard interface for implementing translation tables to work with raw data and
convert them into more readable format of SECTOR/LAYER/COMPONENT. The interface has few methods 
that need to be implemented then the decoder class can handle the translation for the user.
The implementation is simple:

.. code-block:: java

  public class FTCALTranslationTable extends AbsDetectorTranslationTable {

    public FTCALTranslationTable(){
      super("FTCAL",900); // This defines name of the detector and tag=900 for final bank
      // TAG is uded by automated convertor to create event with proper structure
    }

    @Override
    public Integer getSector(int crate, int slot, int channel){
      if(crate==4||crate==5){
        return (crate - 1);
      }
      return -1;
    }

    @Override
    public Integer getLayer(int crate, int slot, int channel){
      if(crate==4||crate==5){
        return (slot*2);
      }
      return -1;
    }

    @Override
    public Integer getComponent(int crate, int slot, int channel){
      if(crate==4||crate==5){
        return channel;
      }
      return -1;
    }
  }


This is a ready class now to be used with decoder class to compile a list of the hits from 
all crated that belong to given detector. 

.. code-block:: java

  import org.jlab.evio.clas12.*;
  import org.jlab.clas12.raw.*;
  import org.jlab.io.decode.*;
  
  ...
  EvioRawEventDecoder    decoder = new  EvioRawEventDecoder();
  FTCALTranslationTable  trTable = new  FTCALTranslationTable();
  List<RawDataEntry> dataEntries = reader.getDataEntries(evioEvent);
  if(dataEntries != null){
    List<RawDataEntry>  ftcalData = decoder.getDecodedData(dataEntries,trTable);
    // The list will contain only entries that were decoded by translation table
  }
  ...


The returned list is a subset of the original list of hits for which translation table
returns getSector(crate,slot,channel) >= 0. Implementation of a new translation table class
is only neccessary in cases when the mapping is not trivial or if the representation of 
translation table in the file will be too large. The abstract translation table class has methods 
to read a translation file and do translation based values from the table. The file format for
table is given as an example:

.. code-block:: bash

  #-----------------------------------------------------------------------------
  # TRANSLATION TABLE
  #-----------------------------------------------------------------------------
  # Detector - Sector - Layer - Component - CRATE - SLOT - CHANNEL
  #-----------------------------------------------------------------------------
  FTCAL      1      1      0      4      3     0
  FTCAL      2      1      0      4      3     1
  FTCAL      3      1      0      4      4     2
  FTCAL      4      1      0      4      4     3
  FTCAL      5      1      0      4      5     4
  ...
  ...

To read this table (say names FTCAL.table) use the standard abstract translation table class:

.. code-block:: java

  ...
  AbsDetectorTranslationTable  trTable = new  AbsDetectorTranslationTable("FTCAL",900);
  trTable.readFile("FTCAL.table");
  System.out.println(trTable); // printout the content of the table
  ...

Writing an output file
======================

Here is a complete code showing how to implement a decoder program for one given detector.

.. code-block:: java

   import org.jlab.evio.clas12.*;
   import org.jlab.clas12.raw.*;
   import org.jlab.io.decode.*;

   String inputFile  = args[0];
   String outputFile = "mydecoded_data.evio";

   AbsDetectorTranslationTable  trTable = new  AbsDetectorTranslationTable("FTCAL",900);

   trTable.readFile("FTCAL.table");
   
   EvioRawEventDecoder    decoder = new  EvioRawEventDecoder();
   EvioRawDataSource       reader = new EvioRawDataSource();
   reader.open(inputFile);

   EvioDataSync  writer = new EvioDataSync();
   writer.open(outputFile);

   while(reader.hasEvent()){
      EvioDataEvent event = reader.getNextEvent();
      List<RawDataEntry> dataEntries = reader.getDataEntries(evioEvent);
      if(dataEntries != null){
        List<RawDataEntry>  ftcalData = decoder.getDecodedData(dataEntries,trTable);
        int nrows = ftcalData.size();
        if(nrows>0){
          EvioDataEvent  outEvent = writer.createEvent(EvioFactory.getDictionary());
          EvioDataBank   bank     = outEvent.getDictionary().createBank("FTCAL::dgtz",nrows);
          for(int loop = 0; loop < nrows; loop++){
             int idx = ftcalData.get(loop).getComponent() % 22;
             int idy = ftcalData.get(loop).getComponent() / 22;
             int TDC = ftcalData.get(loop).getTDC();
             int ADC = ftcalData.get(loop).getADC();
             bank.setInt("idx",loop,idx);
             bank.setInt("idy",loop,idy);
             bank.setInt("ADC",loop,ADC);
             bank.setInt("TDC",loop,TDC);
          }
          outEvent.appendBanks(bank);
          writer.writeEvent(outEvent);
        }
      }
    reader.list(event);
   }

   reader.close();
   writer.close();

This program will read raw hits from all crates, then decoded array will contain only hits from
FTCAL which have entries in the translation table, then these hits will be written into the event.