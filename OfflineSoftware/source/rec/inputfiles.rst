
CLAS12 Data Files
**********************************

CLAS12 uses two data formats for entire chain of data aqcuisition, simulation and reconstruction.
The DAQ produces EVIO files in composite bank structures, with ADC and TDC values for each 
CRATE/SLOT/CHANNEL, which must be decoded and presented in HIPO format for reconstruction to run.
GEMC also presents it's output in EVIO format, which has to be converted into HIPO to run reconstruction.

All examples assume that the coatjava package has been downloaded and unpacked.

Converting RAW data
===================

To convert EVIO files from DAQ to HIPO format, one should use decoder program which decodes raw ADC and
TDC banks, and then applies translation tables to convert CRATE/SLOT/CHANNEL to SECTOR/LAYER/COMPONENT.
Usage:

.. code-block:: bash
   
   >cd coatjava
   >./bin/decoder -o run_012345.hipo run_012345_A00.evio run_012345_A01.evio

This will produce a HIPO file with translated ADC and TDC banks. For compression use :

.. code-block:: bash

   >./bin/decoder -c 2 -o run_012345.hipo run_012345_A00.evio run_012345_A01.evio

The "-c 2" will apply LZ4 compression to the output stream. "-c 1" is used for GZIP compression.
GZIP provides more compact files, though the throughput is lower than for LZ4 compression, so 
recommended compression is LZ4.

Translation of RAW ADC and TDC data to detector component relies on translation tables implemented in
CCDB, if translation table for particular component is not implemented, then data will be written 
to generic bank for ADC and TDC. Here is the structure of untranslated data:

.. code-block:: bash

   {
    "bank": "RAW::adc",
    "group": 801,
    "info": "digitized bank ADC for untranslated",
    "items": [
       {"name":"crate",    "id":1, "type":"int8",  "info":"crate of ADC"},
       {"name":"slot",     "id":2, "type":"int8",  "info":"slot of ADC"},
       {"name":"channel",  "id":3, "type":"int16", "info":"channel of ADC"},
       {"name":"ADC",      "id":4, "type":"int32", "info":"ADC integral from fit"},
       {"name":"time",     "id":5, "type":"float", "info":"time from fitting the pulse"},
       {"name":"ped",      "id":6, "type":"int16", "info":"pedestal from pulse"}
    ]
  },

and for TDC.

.. code-block:: bash

  {
    "bank": "RAW::tdc",
    "group": 802,
    "info": "digitized TDC bank for untranslated entries",
    "items": [
       {"name":"crate",    "id":1, "type":"int8",  "info":"crate of TDC"},
       {"name":"slot",     "id":2, "type":"int8",  "info":"slot for TDC"},
       {"name":"channel",  "id":3, "type":"int16", "info":"channel for TDC"},
       {"name":"TDC",      "id":4, "type":"int32", "info":"TDC value"}
    ]
  }

If desired detector component does not appear in designated bank, they most likely end up
in generic ADC and TDC banks.

Converting GEMC data
====================

To provide uniform input for the reconstruction (wether data comes from DAQ or GEMC), convertion
utility must be used for GEMC files before running reconstruction. To convert GEMC EVIO files to HIPO
use:

.. code-block:: bash

   >cd coatjava
   >./bin/evio2hipo -o gemc_generated.hipo gemc_generated_1.evio gemc_generated_2.evio

This will create a HIPO file where the signals from each detector have been separated into ADC and
TDC banks, similar to DAQ input files. The reconstruction will run on HIPO files starting from 
version 4a.0.0 of coatjava. During conversion the settings for simulation can also be altered.
To override header bank with custom run number and torus/solenoid settings use:

.. code-block:: bash

   >cd coatjava
   >./bin/evio2hipo -r 11 -t -0.5 -s 0.75 -o gemc_generated.hipo gemc_generated_1.evio gemc_generated_2.evio

This will set run number to 11, and torus and solenoid fields to -0.5 and 0.75 respectively.



Reading Data Files
==================

Data produced by decoder and evio2hipo converter have the smae format. For each detector there is 
a bank for ADC, with structure:

.. code-block:: bash

    {"name":"sector",    "id":1, "type":"int8",  "info":"sector of FTOF"},
    {"name":"layer",     "id":2, "type":"int8",  "info":"panel id of FTOF (1-1A, 2-1B, 3-2B"},
    {"name":"component", "id":3, "type":"int16", "info":"paddle id of FTOF"},
    {"name":"order",     "id":4, "type":"int8",  "info":"order of (0 - ADCL , 1 - ADCR)"},       
    {"name":"ADC",       "id":5, "type":"int32", "info":"ADC integral from fit"},
    {"name":"time",      "id":6, "type":"float", "info":"time from fitting the pulse"},
    {"name":"ped",       "id":7, "type":"int16", "info":"pedestal from pulse"}

and for TDC:

.. code-block:: bash

    {"name":"sector",    "id":1, "type":"int8",  "info":"sector of FTOF"},
    {"name":"layer",     "id":2, "type":"int8",  "info":"panel id of FTOF (1-1A, 2-1B, 3-2B"},
    {"name":"component", "id":3, "type":"int16", "info":"paddle id of FTOF"},
    {"name":"order",     "id":4, "type":"int8",  "info":"order of (2 - TDCL , 3 - TDCR)"},
    {"name":"TDC",       "id":5, "type":"int32", "info":"TDC value"}


The banks for raw data are described in etc/bankdefs/hipo/DATA.json (inside the coatjava package).
To read the files use:

.. code-block:: java
   
   import org.jlab.io.hipo.*;

   HipoDataSource reader = new HipoDataSource();
   reader.open("myfile.hipo");

   while(reader.hasEvent()==true){
	DataEvent event = reader.getNextEvent();
	if(event.hasBank("FTOF::adc")==true){
	    DataBank  bank = event.getBank("FTOF::adc");
	    int rows = bank.rows();
	    for(int i = 0; i < rows; i++){
	    	int    sector = bank.getByte("sector",i);
	    	int     layer = bank.getByte("layer",i);
	    	int    paddle = bank.getshort("component",i);
		int       ADC = bank.getInt("ADC",i);
		int     order = bank.getByte("order",i); // order specifies left-right for ADC
		System.out.println("ROW " + i + " SECTOR = " + sector 
				    + " LAYER = " + layer + " PADDLE = "
				    + paddle + " ADC = " + ADC);
	    }
	}
   }

The data structures provided by convertos are the same as for the data distribution ring, and usage
is the same for online data readging. The difference is in using different implementation of data source.
To access online data for monitoring use:

.. code-block:: java

   import org.jlab.io.hipo.*;

   HipoRingSource reader = new HipoRingSource();
   reader.open("129.57.167.60"); // IP address of clondaq5

Either one IP address can be specified or a list of IP addresses separated by ":".
