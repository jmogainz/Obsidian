Controller
	Create controller object
	Initialize controller (config file)
	Initialize message receiver from RemoteService (config file)
	Initialize message receiver from AIOAService (config file)
	Call RemoteService message receive loop in separate thread
		Use multiprocessing queue as argument
	Call AIAOService message receive loop in separate thread
		Use same multiprocessing queue as argument
	Begin controller reactive processing
		Block until valid waypoint message and start command from base station
	Begin controller proactive processing
		Process cycle
			Pure Pursuit Navigation Update to Vehicle Control
			Check Messages Queue
				If ML Message, check what command
					Could be obstacle detected, initiate evasive maneurvers
				If Base Station, check what command
					Could be stop command
RemoteService Commands
	Waypoint List
	Start
	Stop
	Speed
	Shutdown
	
	