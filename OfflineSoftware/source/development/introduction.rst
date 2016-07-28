
Development of Code
*******************

This section describes how start development for CLAS12 detector components code.

Getting Started
===============

Developing in Java done in IDE of users chice. Eclipse or NetBeans can be used.
CLAS12 framework main code uses Maven to distribute and version the software. 
All tools available are packaged in one JAR file for convenience. Separate
libraries can also be found on the Maven repository. Maven repository is set 
up at Jlab : http://clasweb.jlab.org/clas12maven. And the uber JAR (all packed in
one jar) can be found at: http://clasweb.jlab.org/clas12maven/org/jlab/coat/coat-libs.

Two aproaches are available for starting a project, with ant or maven.

Developing with ant
-------------------

Open desired IDE and create a new ant project. Download the JAR file coat-libs 
(desired version), then add it as dependency to your project.

Developing with maven
---------------------

In IDE create a maven project. It will create project directory with 'src' folder
and a pom.xml file. POM files are used to specify dependencies and building rules.
To use coat-libs package in your project you need to modify your pom.xml file as following:

.. code-block:: bash

   <?xml version="1.0" encoding="UTF-8"?>
   <project xmlns="http://maven.apache.org/POM/4.0.0" 
   xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" 
   xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>
   
    <groupId>org.jlab.clas</groupId>
    <artifactId>clasrec-ec</artifactId>
    <version>1.0-SNAPSHOT</version>
    <packaging>jar</packaging>

    <repositories>
      <repository>
        <id>clas12maven</id>
        <url>https://clasweb.jlab.org/clas12maven</url> 
      </repository>
    </repositories>

    <dependencies>
      <dependency>
        <groupId>org.jlab.coat</groupId>
        <artifactId>coat-libs</artifactId>
        <version>3.0-SNAPSHOT</version>
      </dependency>
      <dependency>
        <groupId>org.jlab.coda</groupId>
        <artifactId>jclara</artifactId>
        <version>4.3-SNAPSHOT</version>
      </dependency>
    </dependencies>
    </project>   


The top section describes users project name and version. The dependecy section adds
coat-libs as a dependency to the project, and repositories section provides a link
to CLAS12 maven repository. With this modified POM file the full functionality of
coat libraries is now available.


Calibration Code
================

There are few standard classes developed for user convenience that can be used to
take full advantage of the framework tools. The class DetectorMonitoring can be used
to develop monitoring and calibration software. It provides functionality for loading
calibration constatns and geometries. The user has to implement four abstract methods.

.. code-block:: java

    public void processEvent(EvioDataEvent event); // every event is passed to this
    public void init(); // called at the start of processing
    public void configure(ServiceConfiguration c); // configuration of the class
    public void analyze(); // final analysis (fits and such)


Extending the DetectorMonitoring class also provides a place holder for storing
histograms and analysis results and saving them into a file. Here is a simple
class that will create histograms and fill them with ADC information from 
FTOF counters. The DetectorMonitoring class does not have an empty constructor.
It forces user to set a name, version and author name for the class. This information
is used when dynamically loading classes as plugins.

.. code-block:: java

   public class MyCalibration extends DetectorMonitoring {
   	  public MyCalibration(){
	  	 super("MYCALIB","1.0","developer");
	  }

	  public void init(){
	      TDirectory  myHists = new TDirectory("calibration/ftof1a/TDC");
	      myHists.addObject("HIST_S_1_PADDLE_22",new H1D("HIST_S_1_PADDLE_22",100,0.0,4000.0));
	      getDir().addDirectory(myHist);
	  }

	  public void processEvent(EvioDataEvent event){
	     if(event.hasBank("TFOF1A::dgtz")==true){
		EvioDataBank bank = (EvioDataBank) event.getBank("FTOF1A::dgtz");
		for(int i = 0; i < bank.rows(); i++){
		   if(bank.getInt("sector",i)==1&&bank.getInt("paddle",i)==22){
		   	H1D htdc = (H1D) getDir().getDirectory("calibration/ftof1a/TDC").getObject("HIST_S_1_PADDLE_22");
			htdc.fill(bank.getInt("TDCL",i));
		   }
		}
	     }
	  }
	  
    	  public  void analyze(){
	  	 F1D func = new F1D("gaus",0.0,3000.0);
		 func.setParameter(0,100.0);
		 func.setParameter(1,600.0);
		 func.setParameter(2,50.0);
		 H1D htdc = (H1D) getDir().getDirectory("calibration/ftof1a/TDC").getObject("HIST_S_1_PADDLE_22");
		 htdc.fit(func);
		 func.show(); // prints out parameters of the fit
	  }

	  public abstract void configure(ServiceConfiguration c){}
   }

The code above relies on input generated from GEMC. To run this class through events just add
a main method to the class.

.. code-block:: java

  public static void main(String[] args){
     MyCalibration calib = new MyCalibration();
     calib.init();
     CLASMonitoring monitor = new CLASMonitoring("gemc_output.evio", calib);
     monitor.process();
     calib.analyze();
     TBrowser browser = new TBrowser(calib.getDir()); // opens a GUI to browse histograms
  }

The clas monitoring class loops through events in provided file and calls processEvent method 
for each event on calib class. TBrowser is GUI interface for browsing directory and making 
plots. In some situations the input file is not in standard evio format, and data must be converted
into evio structures that calibration routine can process. Here is an example on how to make
a standard event.

.. code-block:: java

   ...
   MyTextFileParser parser = new MyTextFileParser("datafile.txt");
   while(parser.hasNext()){
	parser.readNext();
	int nrows = parser.rows();
	EvioDataEvent  event = EvioFactory.createEvioEvent();
	EvioDataBank   bankFTOF = (EvioDataBank) event.createBank("FTOF1A::dgtz",nrows);
	for(int loop = 0; loop < nrows; loop++){
		bankFTOF.setInt("sector",loop, parser.getSector(loop));
		bankFTOF.setInt("paddle",loop, parser.getPaddle(loop));
		bankFTOF.setInt("ADCL",loop, parser.getADCL(loop));
		bankFTOF.setInt("ADCR",loop, parser.getADCR(loop));
		bankFTOF.setInt("TDCL",loop, parser.getTDCL(loop));
		bankFTOF.setInt("tDCR",loop, parser.getTDCR(loop));
	}
	event.appendBanks(bankFTOF);
	calib.processEvent(event);
   }
   ...

In this example MyTextFileParser is a user class that parses custom format of data
and creates appropriate bank structures for time of flight 1A panel bank that can 
be processed through calibration code.

Developing for Reconstruction
=============================

The reconstruction code is developed around abstract class DetectorReconsturction,
in many ways it is similar to DetectorMonitoring class (same methods to be extended),
but it also contains code to help with detector geometries and calibration constants.
Same methods need to be implemented for the code to run. The initialization of any external
resources that are independent of configuration have to be done in init() method. Example:

.. code-block:: java

   ...
   public void init(){
   	  requireGeometry("FTOF");
	  requireGeometry("EC");
	  requireCalibration("FTOF");
   }
   ...

This will initialize the FTOF and EC geometries and calibration constants to be used in the code.
Somewhere in the code processEvent() one can use the preloaded geometry.

.. code-block:: java
   
   public void processEvent(EvioDataEvent event){
       ...
       ECSector sector = (ECSector) getGeometry("EC").getSector(0);       
       ...
   }

More on how to use the geometry package refer to section "Geometry Package".
