
Analyzing Reconstructed Data
****************************

This section describes analysis tools available within clara framework
to analize reconstructed data events.

Groovy Scripts
==============

The analysis can be run as groovy scripts, and all Java classes can be used
inside groovy script. This method ties well with the framework and provides
standard tools for reading EVIO files and reconstruction banks. To run groovy
scripts the groovy package has to be installed on the computer. On OSX machines
this can be done by:

.. code-block:: bash
   
   > sudo port install groovy

On Jlab farm machines one has to initialize groovy by:

.. code-block:: bash

   > use groovy

Once groovy is installed on the system any groovy script can be run using:

.. code-block:: bash

   coatjava> bin/run-groovy myscript.groovy

The example scripts are located in scripts folder in the distribution.



Reading EVIO files
==================

The output of reconstruction is an EVIO file with bank structures defined in XML 
files. The XML descriptors are located in etc/bankdef/clas12 directory. Following 
script will run through events in the file:

.. code-block:: java
    
       import org.jlab.io.evio.*;
       
       EvioSource reader = new EvioSource();
       reader.open("myfile.evio");
       while(reader.hasEvent()){
	  EvioDataEvent event = reader.getNextEvent();
	  event.show(); // print out all banks in the event
       }

The banks are reffered to with their name and the section separated by '::',
for example to read digitized bank from DC the following code can be used:

.. code-block:: java

   ...
   EvioDataEvent event = reader.getNextEvent();
   if(event.hasBank("DC::dgtz")){
      EvioDataBank dcBank = event.getBank("DC::dgtz");
      dcBank.show(); // printout the content of the bank
   }
   ...

The EvioDataSource object is derived from DataSource interface, and there are several implementations
of DataSource, one of them is the BOS (legacy CLAS6 format) reader. In the same manner the BOS files
can be read within our Java framework with no additional effort. To read bos files the bos reader 
interface can be used. There are slight differences in the bank description in BOS format, since the
BOS banks are numbered, the descriptor to reatrieve particular bank should contain the number of the 
bank within the name, if no number is present '0' will be assumed.


Reading Reconstruction Events
=============================

The EVIO structures produced by reconstruction program can be parsed to produce
events for physics analysis. There is a standard interface allowing to implement
different ways to identify particles. One simple implementation exists called
GenericKinematicFitter which reads generated particle bank and the reconstructed
particle bank with the PID assigned by standrard Event Builder. Example:


.. code-block:: java

   import org.jlab.io.evio.*;
   import org.jlab.clas.physics.*;

   EvioSource reader = new EvioSource();
   reader.open("myrec.evio");
   // create new kinematic fitter, with beam energy 11.0 GeV and electron filter
   GenericKinematicFitter fitter = new GenericKinematicFitter(11.0);
   
   while(reader.hasEvent()==true){
	EvioDataEvent event = reader.getNextEvent();
        PhysicsEvent  recEvent  = fitter.getPhysicsEvent(event);
        PhysicsEvent  genEvent  = fitter.getGeneratedEvent(event);
	System.out.println(genEvent.toLundString());
	System.out.println(recEvent.toLundString());
   }
   
This will print out on the screen generated event and reconstructed event.

Working with Physics Events
===========================

Physics event object is a container for particles and has a beam particle and 
a target particle, there are few convenience methods that alow checking the 
final state of the event. Particle id's are used to require specific particle
to be in the final state, and "X" followed by the sign is used to require any 
number of particles with particular charge. For example "X+" means any number 
of positively charged particles, "X-" and "Xn" for negative and neutral particles
respectively. EventFilter object must be created to check the final state
of the event.

.. code-block:: java

   ...
   EventFilter  filter = new EventFilter("11:2212:-211:X+:X-:Xn");
   if(filter.isValid(recEvent)==true){
	Particle mx_eppi = recEvent.getParticle("[b]+[t]-[11]-[2212]-[-211]");
	double mass  = mx_eppi.mass();
	double mom   = mx_eppi.p();
	double theta = mx_eppi.theta();
	double phi   = mx_eppi.phi();
   }
   ...

The code above checks if the event has at least one electron, one proton and one 
negative pion and then constructs a missing mass of "e-,p,pi-". The symbol "[b]"
stands for beam particle and "[t]" stands for target particle. There can be multiple
particle of the same kind in the event, and the syntax allows picking up particles
by the order. For example:

.. code-block:: java
   
   ...
   EventFilter  filter = new EventFilter("11:2212:2212"); // exclusive e-,p,p
   if(filter.isValid(recEvent)==true){
	Particle mx_epp = recEvent.getParticle("[b]+[t]-[11]-[2212,0]-[2212,1]");
   }
   ...

Entry "[2212,1]" takes the second (skip=1) proton from the event, "[2212,0]" takes the first
event, if no skip parameter is mentioned first particle is assumed "[2212]" is same 
as "[2212,0]".
The following example loops through events and plots the missing mass of two pions.

.. code-block:: java

   import org.jlab.io.evio.*;
   import org.jlab.groot.data.*;
   import org.jlab.groot.ui.*;
   import org.jlab.clas.physics.*;

   EvioSource reader = new EvioSource();
   reader.open("myrec.evio");
   GenericKinematicFitter fitter = new GenericKinematicFitter(11.0);
   EventFilter  filter = new EventFilter("11:211:-211:X+:X-:Xn");
   H1F MxPiPi = new H1F("MxPiPi",120,0.01,0.35);

   while(reader.hasEvent()==true){
     EvioDataEvent event = reader.getNextEvent();
     PhysicsEvent  recEvent  = fitter.getPhysicsEvent(event);

     if(filter.isValid(recEvent)==true){
        Particle mx_epipi = recEvent.getParticle("[b]+[t]-[11]-[211]-[-211]");
        MxPiPi.fill(mx_epipi.mass());
     }
   }

   TCanvas c1 = new TCanvas("c1",1000,800);
   c1.divide(1,1);
   c1.cd(0);
   c1.draw(MxPiPi);

The code loops through events and picks events corresponding to the given filter
then gets particle for given string syntax and fills the mass histogram.


Selecting particles
===================

PhysicsEvent object can be used to construct composite particles from the final sate 
(as in the examples above), but it can alse be used to loop through particles with
either given charge or given pid. Example:

.. code-block:: java

   ...
   int nparticles = recEvent.count(); // returns number of particles
   int nphotons   = recEvent.countByPid(22); // returns number of photons
   int nneutrals  = recEvent.countByCharge(0); // return number of particles with charge=0
   int npositive  = recEvent.countByCharge(1); // return number of particles with charge=+1
   ...
   if(nphotons>=2){
      Particle  photon1 = recEvent.getParticleByPid(22,0); // skip=0, returns first photon
      Particle  photon2 = recEvent.getParticleByPid(22,1); // skip=1, skips first and returns second photon
      System.out.println(" mom 1 = " + photon1.mass() + "  mom 2 = " + photon2.p());
   }
   ...
   for(int loop = 0; loop < npositive; loop++){
      Particle posPart = recEvent.getParticleByCharge(1,loop); // skips loop particles to return the next one
      if(posPart.pid()==211){
	System.out.println(" found pion with angle = " + posPart.theta());
      }
   }
   ...

The loop over all particles can also be made by checking their pid:

.. code-block:: java

   ...
   int nparticles = recEvent.count(); // returns number of particles
   for(int loop=0; loop < nparticles; loop++){
      Particle part = recEvent.getParticle(loop);
      if(part.pid()==2212){
	System.out.println(" found proton in row " + loop);
	System.out.println(" vertex Z = " + part.vertex().z() );
      }
   }
   ...

Working with Particles
======================

The implementation of Particle class provides operations with particles, such as combine them,
calculating vertex or matching particles. (Examples SOON).
