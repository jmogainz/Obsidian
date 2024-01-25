Range [Direct Access]
Bearing [Direct Access]
Altitude [Direct Access]
Elevation [Direct Access]
Velocity [Direct Access]
Veritcal Velo [Calculated during Guardian Classification]
Range-rate [Direct Access]
Signal-to-noise ratio? [Direct Access]
Frequency [Direct Access]
Pulsewidth [SignalList->Signal[i].mPulseWidth] (cant rule out yet, but PUT ON HOLD)
Pulse repetition interval [SignalList->Signal[i].mRepetitionInterval]
Covariance matrix [Direct Access]

# BUILD RELEASE
# CHECK ALL CONFIGS
# LOGICAL TIME = NO LOGGING

Building Afsim for Gaurdian:
	Move unbuilt plugins into swdev/src/wsf_plugins 
		Fire_control, sensor_observer, time_controller
	Build to vs sln, compile build, compile install
		Install folder: build/wsf_install
	Move pre-built plugins into build/wsf_install/bin ()
		Helmsdeep [check]

Guardian main folder:
 	Modified:
		config.AFSIMbridge.json (my system paths)
		platform file for range
		batctest.guard2.json
		
Dev Iteration STEPS:
1. Build toolbox release
2. Build guardian release
3. Make sure plugins are in afsim
4. Build afsim release
5. Run shortcut_HD_placement.py (if fresh)
6. Modify batchtest.guard2.json if need be (logical)
7. Modify platform file if need be
8. Modify Config.batchcontroller.json
9. Modify afsimSceneGeneratorConfig folder (nvm)
10. Modify guardian_scenario_manual_fire_HD.txt
11. Run laydown file .py generator
12. Comment out RECORDER in .bat starter
13. Run RabbitMQ server
14. guardianstartforbatch.bat -logging debug
15.  UDP ON BOTH, -environ
16. Batchcontroller.exe -runtest Guard2

Gaurdian install folder:
	Modify Install:
		afsimSceneGeneratorConfig folder [check]
		Batchtest.guard2.json [check]
		Config.batchcontroller.json [check]
		Maybe the recorder
		Guardian_scenario_controller_ml_base.txt [check]
		platform_mshorad_manual_fire_HD.txt [pending]
		Place helmsdeep jni folder in scenarios [check]
	Generate random scenario files [check]
	Run:
		Start RabbitMQ [check]
		Start Gaurdian Services, with logging [] <debug>
		Batchcontroller.exe -runtest Guard2
	Change code:
		Plugins
			
For DEV:
Open afsim.sln (where changes will be made) (then rebuild)
Open toolbox.sln (reference) (changes made to plugins need to be moved here)
Open guardian.sln (reference) (changes made to plugins need to moved here)

ERRORS:
	On Windows, toolbox, guardian, and afsim must be build with 'release'
	Must run logical time no logging
	
TODOS:
	TODO: Bringing in range required far too much code alteration, restructure for faster data extraction
		Ideas | string-to-feature map, 
	TODO: Leave Jarrod Status Update Friday
	
	
RECENT RECENT:
	
	Increased time wait constants in AfsimBridge
	Put debugging statements in TimeControl plugin
	
	Everything else latest is committed to ML_DEV branch
	

	
