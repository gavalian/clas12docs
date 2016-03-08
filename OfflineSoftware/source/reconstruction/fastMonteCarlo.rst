
CLAS12 Fast MonteCarlo
**********************

The CLAS12 fast Monte-Carlo is based on full detector simulation and
particles are swam through magnetic field (Solenoid and Torus).

Running From Lund File
======================

The event reconstruction is automated in fast MC, and physics events can be passed
through detector, returning a PhysicsEvent object with particles that passed through
detector and enough hits were generated in Drift Chambers, Calorimeter, Time Of Flight
counters and others depending on weather the particle goes through forward detector
or central detector.
Here is a simple script that will run events from Lund file and printout events
where the electron was detected in the detector.

.. code-block:: java

   import org.jlab.clasrec.io.*;
   import org.jlab.clas.physics.*;
   import org.jlab.clas.reactions.*;
   import org.jlab.clas.physics.*;
   import org.jlab.clas12.fastmc.*;

   String inputFile = args[0];

   CLAS12FastMC    fastMC = new CLAS12FastMC(-1.0,1.0); // Torus = -1, Solenoid = 1.0

   LundReader      reader = new LundReader();
   reader.addFile(inputFile);
   reader.open();

   while(reader.next()==true){
	PhysicsEvent genEvent = reader.getEvent();
   	PhysicsEvent recEvent = fastMC.getEvent(genEvent);
   	if(recEvent.countByPid(11)>0){
      	   System.out.println("==============>  EVENT PRINTOUT");
           System.out.println("----->  GENERATED EVENT");
           System.out.println(genEvent.toLundString());
           System.out.println("----->  RECONSTRUCTED EVENT");
           System.out.println(recEvent.toLundString());   
       }
   }

The output:

.. code-block:: bash

   ==============>  EVENT PRINTOUT
   ----->  GENERATED EVENT
           6  1.  1.  1  1 0.000   0.000   5.017   0.000   0.000
    1 -1.    1     11  0  0    0.9689   -0.2909    3.9055    4.0344    0.0005      0.0000    0.0000    0.0000
    2 -1.    1   -211  0  0    0.3342    0.1975    0.0290    0.4135    0.1396      0.0000    0.0000    0.0000
    3  0.    1     22  0  0   -0.5877    0.0430    2.5465    2.6138    0.0000     -0.0000   -0.0000    0.0000
    4  0.    1     22  0  0   -0.5472   -0.0903    2.3951    2.4585    0.0000     -0.0000   -0.0000    0.0000
    5  1.    1   2212  0  0   -0.0779    0.2248    1.8261    2.0668    0.9383      0.0000    0.0000    0.0000
    6  1.    1    211  0  0   -0.0904   -0.0841    0.2978    0.3513    0.1396      0.0000    0.0000    0.0000

   ----->  RECONSTRUCTED EVENT
           3  1.  1.  1  1 0.000   0.000   5.017   0.000   0.000
    1 -1.    1     11  0  0    0.9689   -0.2909    3.9055    4.0344    0.0005      0.0000    0.0000    0.0000
    2  0.    1     22  0  0   -0.5877    0.0430    2.5465    2.6138    0.0000     -0.0000   -0.0000    0.0000
    3  0.    1     22  0  0   -0.5472   -0.0903    2.3951    2.4585    0.0000     -0.0000   -0.0000    0.0000

Running with Particle Generators
================================

The fast MC class can also analyze individual particles for acceptance. In the example below we generate
random electron with given energy and agular constrains. Then we pass it to fast MC class, if returned
particle had momentum =0, then particle was not reconstructed by detector:

.. code-block:: java

  import org.jlab.clasrec.io.*;
  import org.jlab.clas.physics.*;
  import org.jlab.clas.reactions.*;
  import org.jlab.clas.physics.*;
  import org.jlab.clas12.fastmc.*;
  import org.jlab.geom.prim.*;
  import org.jlab.geom.*;

  ParticleGenerator  electron = new ParticleGenerator(  11 , // electron PID  
                             1.0  ,    5.0, // momentum min/max 
                             10.0 ,   125.0, // theta (deg) min/max
                          -180.0  ,  180.0 ); // phi (deg) min/max


  CLAS12FastMC    fastMC = new CLAS12FastMC(-1.0,1.0);

  int countrec = 0;
  int count    = 0;
  for(int loop = 0; loop < 100; loop++){
    Particle ep = electron.getParticle();
    Particle eprec = fastMC.getParticle(ep);
    count++;
    if(eprec.p()>0) countrec++;
    System.out.println("=========>  Event Printout: ");
    System.out.println(" GEN : " + ep.toLundString());
    System.out.println(" REC : " + eprec.toLundString());
  }

  System.out.println("\n\n Reconstructed event " + countrec + "/" + count);


In general to construct a particle and test it in fast MC, the following code is needed:

.. code-block:: java

  Particle myP = new Particle(2212, 0.5, 0.6, 1.7); // pid = 2212, px/py/pz = 0.5 0.6 1.7
  // or with vertex new Particle(2212, 0.5, 0.6, 1.7, 0.0, 0.0, -35.0);

  Particle myPRec = fastMC.getParticle(myP);
  if(myPRec.p()>0){
    System.out.println("reconstructed");
  } else {
    System.out.println("not even close");
  }


Adding Resolutions to Reconstructed Particles
=============================================

The code changes properties of reconstructed particles (such as momenta and angle)
through IParticleResolution interface. User supplies the implementation of the code
to Fast Monte Carlo for specific PID, the rest is done automatically by fast MC code.
Notice and additional import statement in the code (import org.jlab.clas.pdg.*;).
Here is an example of how it can be used:

.. code-block:: java

  //****************************************************
  // Particle generator with decay module
  //****************************************************
  import org.jlab.clasrec.io.*;
  import org.jlab.clas.physics.*;
  import org.jlab.clas.reactions.*;
  import org.jlab.clas.physics.*;
  import org.jlab.clas.pdg.*;
  import org.jlab.clas12.fastmc.*;
  import org.jlab.geom.prim.*;
  import org.jlab.geom.*;

  ParticleGenerator  electron = new ParticleGenerator(  11 , // electron PID  
               1.0  ,    5.0, // momentum min/max 
               7.0  ,   25.0, // theta (deg) min/max
          -180.0  ,  180.0 ); // phi (deg) min/max


  CLAS12FastMC    fastMC = new CLAS12FastMC(-1.0,0.5);
  ParticleSwimmer pswim = new ParticleSwimmer();

  // This part implements an interface for callback on particle
  // type 11 (electron), and it will be called whenever any electron
  // passes the detectors cuts and appears in reconstructed event
  // it's momenta will be changed with gaussian distribution of 2%
  fastMC.addResolutionFunc( 11,new IParticleResolution(){

    @Override
    void apply(Particle p){
      double mom = p.p();
      double fraction = PhysicsConstants.getRandomGauss(0.0,0.02);
      p.setP(mom + mom*fraction);
    }

  });

  PhysicsEvent  genEvent = new PhysicsEvent();

  for(int loop = 0; loop < 20; loop++){
    
    Particle ep = electron.getParticle();
    genEvent.clear();
    genEvent.addParticle(ep);

    PhysicsEvent recEvent = fastMC.getEvent(genEvent);

    System.out.println("\n\n------> event # " + loop);
    System.out.println(genEvent.toLundString());
    System.out.println(recEvent.toLundString());

  }


