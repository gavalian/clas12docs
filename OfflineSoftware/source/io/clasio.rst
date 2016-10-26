
.. _clasio-reading:

Reading/Writing Dictionary based files
**************************************

CLAS/IO library is a java package developed to handle input/output of the 
common data types used in CLAS. The interface is implemented to make codes
trasparent to the different types of data used. Implementations exist for
formats BOS/FPACK, EVIO (clas12), and iG5 (data mining format). The document 
describes usage of the package with Jython.


CLAS12 Evio
===========

Working with Dictionary
-----------------------

The bank definitions for CLAS12 are stored in XML files and are dynamically loaded
at run time using the environmental variable CLAS12DIR or CLARA_SERVICES, if either
of these variables are set the program will look for directory lib/bankdefs/clas12
relative path for XML files to load.

Each XML file contains several bank definitions for one detector including raw generated
banks and reconstructed bank structures. Each bank is divided into sections, and each section
has variables with same "tag" number and changing "num".
Here is a sample XML describing a bank with two sections:

.. code-block:: python

   <evio_dictionary>
   <bank name="CUSTOM" tag="100" info="custom bank">
   <section name="part" tag="101"  num="0"   info="bank rows">
    <column name="status"  type="int8"    num="1"   info="status of the particle"/>
    <column name="charge"  type="int16"   num="2"   info="charge of the particle"/>
    <column name="pid"     type="int32"   num="3"   info="particle ID"/>
    <column name="px"      type="float32" num="7"   info="x component of the momentum"/>
    <column name="py"      type="float32" num="8"   info="y component of the momentum"/>
    <column name="pz"      type="float32" num="9"   info="z component of the momentum"/>
    <column name="vx"      type="float32" num="10"  info="x coordinate of vertex"/>
    <column name="vy"      type="float32" num="11"  info="y coordinate of vertex"/>
    <column name="vz"      type="float32" num="12"  info="z coordinate of vertex"/>
  </section>
  <section name="detector" tag="102" num="0" info="detector information">
    <column name="detectorid"  type="int8"     num="1"   info="detector id"/>
    <column name="pindex"      type="int8"     num="2"   info="index to particle section"/>
    <column name="X"           type="float32"  num="3"   info="X position of detector hit"/>
    <column name="Y"           type="float32"  num="4"   info="Y position of detector hit"/>
    <column name="Z"           type="float32"  num="5"   info="Z position of detector hit"/>
    <column name="energy"      type="float32"  num="6"   info="energy deposited in detector"/>
    <column name="time"        type="float32"  num="7"   info="time (ns)"/>
  </section>
 </bank>
 </evio_dictionary>


Each section of the bank will be presented as separate bank descriptor. This file will be parsed to have
two descriptors: "CUSTOM::part" and "CUSTOM::detector". Here is an example groovy script showing parsing of the
file and displaying bank information.

.. code-block:: java
   
   import org.jlab.io.evio.*;
   
   EvioFactory e = new EvioFactory();

   e.loadDictionary(".");
   e.getDictionary().show();

The factory is initialized with XML files found in current directory and the content of 
the dictionary is displayed. The output is:

.. code-block:: bash
   
   +------------------------------------------+--------+--------+--------+
   |                                      Bank| Columns|     Tag|  Number|
   +------------------------------------------+--------+--------+--------+
   |                          CUSTOM::detector|       7|     101|       0|
   |                              CUSTOM::part|       9|     102|       0|
   +------------------------------------------+--------+--------+--------+


Two bank descriptors were initialized, each with three columns and 101 and 102 tags respectively.
Initialization of the dictionary can be done in several ways. The method loadDictionary() with
no arguments will search for XML files in directory $CLAS12DIR/lib/bankdefs/clas12, and will
parse all XML files in that directory into descriptors. The method loadDictionary(String dirpath),
will search for XML filed in given directory path.

Show method prints out dictionary descriptors information, for more detailed information about 
particular bank descriptor, use:

.. code-block:: java

   e.getDictionary().getDescriptor("CUSTOM::detector").show();

Output:

.. code-block:: bash

        +------------------------+--------+--------+------------+
        |                  Column|     Tag|  Number|        Type|
        +------------------------+--------+--------+------------+
        |              detectorid|     102|       1|        int8|
        |                  pindex|     102|       2|        int8|
        |                       X|     102|       3|     float32|
        |                       Y|     102|       4|     float32|
        |                       Z|     102|       5|     float32|
        |                  energy|     102|       6|     float32|
        |                    time|     102|       7|     float32|
        +------------------------+--------+--------+------------+


This will print out detailed information for given descriptor. The first line
gives the descriptor name and the parent container tag (102 in this case), and the
table describes each column with name, tag, number and data type.

Creating Banks
--------------

Once the dictionary has been initialized user can create banks for given structures. Created bank will be 
initialized with given size. Example:

.. code-block:: java

   EvioDataBank   bank = (EvioDataBank) EvioFactory.getDictionary().createBank("CUSTOM::detector",2);
   bank.show();

This will create an instance of "CUSTOM::detector" bank and will allocate 2 rows for each column. And the show()
method of the bank will printout the content of the bank with column names. Output looks like:

.. code-block:: bash
   
        pindex :             0              0  
    detectorid :             0              0  
          time :       0.00000        0.00000  
             Y :       0.00000        0.00000  
             X :       0.00000        0.00000  
        energy :       0.00000        0.00000  
             Z :       0.00000        0.00000 

Newly initialized bank has all entries equal to zero. To modify the entries set<type>() functionas are used.
setFloat(name, row, value) or setInt(name, row, value). Here is an example:

.. code-block::	java

  bank.setByte("detectorid", 0, (byte) 15);
  bank.setByte("pindex",     0, (byte)  1);
  bank.setFloat("X",         0,  2.34);
  bank.setFloat("Y",         0,  3.45);
  bank.setFloat("Y",         0,  4.78);
  bank.setFloat("time",      0,  0.34);
  bank.setFloat("energy",    0,  1.23);
  bank.show();

Output:

.. code-block:: bash
   
  *****>>>>> BANK CUSTOM::detector  >>>> SIZE = 7
        pindex :             1              0  
    detectorid :            15              0  
          time :       0.34000        0.00000  
             Y :       4.78000        0.00000  
             X :       2.34000        0.00000  
        energy :       1.23000        0.00000  
             Z :       0.00000        0.00000 


Writing Events to a file
-------------------------

The class EvioDataSync is used to write events into an EVIO file. The following script will
write 10 events into a newly created file.

.. code-block:: java

 EvioDataSync  writer = new EvioDataSync();
 writer.open("myfirstfile.evio");

 for(int loop = 0; loop < 10; loop++){
  EvioDataEvent event = (EvioDataEvent) writer.createEvent();
  EvioDataBank   bankDET = (EvioDataBank) EvioFactory.getDictionary().createBank("CUSTOM::detector",2);
  EvioDataBank   bankPRT = (EvioDataBank) EvioFactory.getDictionary().createBank("CUSTOM::part",1);
  // Fill detector bank 
  bankDET.setByte("detectorid", 0, (byte) 15);
  bankDET.setByte("pindex",     0, (byte)  1);
  bankDET.setFloat("X",         0,  2.34);
  bankDET.setFloat("Y",         0,  3.45);
  bankDET.setFloat("Y",         0,  4.78);
  bankDET.setFloat("time",      0,  0.34);
  bankDET.setFloat("energy",    0,  1.23);
  // Fill particle bank
  bankPRT.setByte ("status",     0, (byte) 2);
  bankPRT.setShort("charge",     0, (short) +1);
  bankPRT.setInt  ("pid"   ,     0, 2212);
  bankPRT.setFloat("px",    0,  0.489 );
  bankPRT.setFloat("py",    0,  0.703 );
  bankPRT.setFloat("pz",    0,  0.982 );
  event.appendBanks(bankPRT,bankDET);
  writer.writeEvent(event);
 }
 writer.close();

The appendBanks takes any number of arguments, all the banks passed to the method have to be sections of the same bank.

Reading Events From a file
--------------------------

For now the evio files do not contain the dictionary object, in the future the dictionary XML
file will be enbedded in the file so user does not have to worry about having the XML descriptors 
locally to be able to access the banks. For now, the EvioSource object has to know where the 
XML files with bank descriptions are located. The dafault directory is $CLAS12DIR/lib/bankdefs/clas12.
If there is a custom bank dictionary is used to write the file, it must be initialized 
before reading the file. In this example we need first to load the CUSTOM.xml file by loading the 
dictionary from current directory.

.. code-block:: java
   
  import org.jlab.evio.clas12.*;

  EvioFactory.resetDictionary();
  EvioFactory.loadDictionary(".");
  EvioFactory.getDictionary().show();

  EvioSource  reader = new EvioSource();
  reader.open("myfirstfile.0.evio");

  while(reader.hasEvent()==true){
    EvioDataEvent event = (EvioDataEvent) reader.getNextEvent();
    if(event.hasBank("CUSTOM::detector")==true){
       EvioDataBank  bank = (EvioDataBank) event.getBank("CUSTOM::detector");
       bank.show();
    }
  }

This script will read the file created by previous script and printout the CUSTOM::detector bank if
it exists in the event.
