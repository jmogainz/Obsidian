AIAO
	Create AIAO object
	Initialize AIAO (config file)
	Initialize message receiver from ControlService (config file)
	(Possibly) - Initialize message receiver from Prediction Server
	Call AIAO reactive processing
		Block until green light from ControlService
	Call AIAO proactive processing
		Check ControlService message queue for commands
		Retrieve realtime lidar/rgb data
		Send to prediction server/ process and predict internally
		Block until prediction response in message receiver

ControlService Commands
	Start
	Stop
	Shutdown
	