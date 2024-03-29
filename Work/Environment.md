- Sets up everything needed for a single instance of our reinforcement learning environment to run independent of other instances
	- Launches services to run afsim and get data back out of afsim
		- Run afsim: AfsimBridge launches afsim and managers the DIS communications out (which go to position), and the sequential timestepping of afsim
		- Get data back out to our environment (observation)
			- PositionService: Receives data from bridge concerning the dis position of all platforms present in current afsim scenario
			- TrackService: Receives data directly from toolbox sensor observer plugin inside of afsim 
	- Generates afsim ports configuration so that afsim can communicate to our services independent of other instances of the environment
	- These things could/would need to be altered to work with a different environment sim
	- The only other thing left to do to actually kick off afsim is for batchcontroller to tell afsimBridge to do so (batchcontroller is our scenariomanagement that facilities synchonization between all off the services with afsim, and it also can run multiple scenarios sequentially)
- Facilitates all of the reinforcement learning parts of our environment
	- Takes steps
		- Send actions into afsim
		- Receives observations back from afsim
		- Calculates rewards
	- Resets environment when episode/timestep/experience is complete
		- Resets our track service repo managers
		- Resets our positions service repo managers
		- Resets our track repos
		- Resets our position repos
		- Resets barrier
- How we are able to hold up BatchController to not advance the sim clock until our action has been taken and we have gotten an observation back.
	- Condition variable synchonization between batchcontrollers advancelogical time and environments step function
- We have pybinded step and reset back to the environment.py
- JSON
	- Command Readers (broadcast and regular) initialized in Service cpp
	- Command Reply Writer initialized in Service cpp
	- PositionRepository - PositionReader - initialized in environment cpp
	- TrackRepository - TrackReader - initialized in environment cpp
		- TrackSource = SystemTrack (Track Server data instead of DIS)
- Toolbox Service
- Has a lifecycle handler for start and stop reactive and proactive
- Has an options handler for optionChanged (currently we do not have any options)
- Has a timestep handler for prepare and advance logical time (this is what allows us to hold up batchcontroller), batchcontroller (scenariomanager in bc will send out an advance to all its broadcaster writer ports), will not advance to the next time step until everyone else has
- Has a command recipient (simmessagerecipient) for extra receiveMessage processing if need
- temporalFluxCapacitorConditionVariable_ is to ensure that initialize does not return for batchcontroller to start until the service (temporalfluxcapacitor) is up and running (registered to receive advancelogicals for example)
	- TrackRepo and PositionRepo get up and running as well
- The barrier (initialized with a count of 2) ensures that the environment instances associated main python thread is in sync with batchcontroller specifically at the start of the scenario, batchcontroller will not be able to start the scenario and leave the environment behind until step is called from the main python thread
		