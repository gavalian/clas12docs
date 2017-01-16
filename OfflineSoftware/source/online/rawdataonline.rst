
.. _clasio-raw:

**********************
Online Data Monitoring
**********************

COAT-JAVA package contains tools for online data monitoring. It relies on
xMsg (ZeroMQ) data ring distribution system, which works off of ET ring or a file.

Creating xMsg ring from file
============================

xMsg ring works as a deocder service for raw evio data and provides Hipo event with decoded
CLAS12 data. to start the ring use command:

.. code-block:: bash

  >./bin/x-server -type evio -file sector2_000233_mode7.evio.0

This will start Hipo decoded event ring. the file will be continuisly lopped through. It is 
good for testing monitoring software. Following sections will describe how to connect to the 
service and receive decoded data. 

.. note:: The EVIO file must be raw DAQ file. Otherwise empty events will be returned.

Creating xMsg ring from	ET
==========================

Similarly xMsg ring can be started from online ET ring, which will get raw EVIO events from DAQ
ring and distribute them to client programs. To initiate an ET ring distribution service use command:

.. code-block::	    bash

  >./bin/x-server -type et -file /tmp/et_sys_clasprod

The program must run on the same machine where the ET ring is running (no remote access is available).
But the connection to xMsg Hipo ring can be done from any machine within the network. To connect to
remote ET ring. use:

.. code-block::     bash

  >./bin/x-server -type et -host adcecal3 -file /tmp/et_sys_clasprod2

The xMsg ring distribution system publishes two different streams, one for decoded HIPO data type, and
one for raw EVIO data event, which contains composite banks with ADC pulse. User program can subscribe
to any of these streams, to read the data by using HipoRingSource or EvioRingSource class. Both these
classes are implementations of DataSource interface and work as disk files.

Creating xMsg ring from HIPO files
==================================

A event transport ring can be also emulated from HIPO files. If the type
HIPO is used, the events from HIPO file are passed around without any 
conversion or decoding. To start a ring from HIPO file use:

.. code-block::     bash

  >./bin/x-server -type hipo -file my_simple_file.hipo

To use HIPO ring, EVIO file first has to be converted to HIPO, then
run as a HIPO ring. For DAQ EVIO files, first:

.. code-block::     bash

  >./bin/decoder -o my_simple_file.hipo clasrun_012345.evio.0

Then use the HIPO file as a source of a event transport ring.
If using GEMC generated file, then first convert it:

.. code-block::     bash

  >./bin/evio2hipo -o my_simple_file.hipo gemc_dis.evio

Then use x-server utility to run a HIPO event transport ring.


Connecting to xMsg ring (HIPO)
==============================

Connecting to xMsg distribution system is done using standard CLAS12 io interface. Here is code sample
that can connect to specified server and start receving decoded events:

.. code-block:: java

   import org.jlab.io.hipo.*;
   
   HipoRingSource reader = new HipoRingSource();
   reader.open("129.57.167.227");
   while(true){
	if(reader.hasEvent()==true){
	   DataEvent event = reader.getNextEvent();
	   event.show();
	}
   }

This code will attempt to make connection to xMsg ring running specified IP address. 
If the address of ring is not known, a range of addresses can be provided to the open()
method to explore all of them, and it will connect to first available ring. for exmaple.

.. code-block::	  java

   HipoRingSource reader = new HipoRingSource();
   reader.open("129.57.167.227:129.57.167.127:129.57.167.163:129.57.167.101");

This code will try to connect to all specified IP addresses in a given order. First available
xMsg ring will be connected to. For conveniance, there are static methods for connecting
to online machines. The IP addresses of all known online DAQ machines are checked for connection.
To connect to counting house online machines one could use:

.. code-block:: java

   import org.jlab.io.hipo.*;

   HipoRingSource reader = HipoRingSource.createSourceDaq();

This will examine all DAQ machines for active running xMsg data distribution ring, and estabilish
connection with first available machine.

Connecting to xMsg ring (EVIO)
==============================

To connect to raw EVIO stream a EvioRingSource class can be used.

.. code-block:: java

   import org.jlab.io.hipo.*;
   
   EvioRingSource reader = new EvioRingSource();
   reader.open("129.57.167.227");
    while(true){
      if(reader.hasEvent()==true){
        DataEvent event = reader.getNextEvent();
        event.show();
    }
   }

EVIO events from data ring have to be decoded and translation table has to be applied.


