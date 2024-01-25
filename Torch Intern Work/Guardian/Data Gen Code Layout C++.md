TOP -> BOTTOM (Guardian, Begin: ClassificationApp.cpp)
	saveTrainingToDisk
	Classification Features
		double speed;
		double altitude;
		double verticalSpeed;
	extractTrack[whatever]Feature(SystemTrack track)
		Vertical velo is calculated (using prior and current positions)
	Generatetrainingdata
	SystemTrack.Get[whatever] ()
	Receive trackreport and parse into system track (SimRecipient constantly open and receiving)
		Trackrepository (using trackreceiver)
			TrackReceiver: public SimMessageRecipient
				receiveMessage
				handletrackupdated
				classifyTrack_timer
TOP -> BOTTOM (Toolbox, Begin: TrackService.cpp)
	sendTrackReport
		build buffer message for trackreportmessage
	trackit->second->set[whatever] () OR SystemTrack Constructor in handlesourcetrackmessage
	sensorTrackMessage.get[whatever] ()
		return [whatever]ECEF;
	SensorTrackMessage Constructor(with message buffer)
		decodes [whatever] from buffer
	SensorTrackReceiver
		receive message from SimMessageRecipient
		
BOTTOM -> TOP (Asfim, Begin: SensorTrackMessage.cpp)
reports_[whatever]
SensorTrack[whatever] = std::array<double, 3>;
SensorTrackMessage Constructor(with argument values) in build message
buffer = messageToSend.converttoBuffer() in SensorTrackUpdated




	
	
	