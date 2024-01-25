flushReaders()
	- used on shutdown 
	- used on stopreactiveprocessing
	
stopreactiveprocessing() for classification service
	- writing to disk!
	
How can we stop shutting down prematurely?
	THE SERVICE is shutting down
	
We know:
	ClassificationMain
		Create Classification Service obj - service
		Load in config data - configData
		Create ClassificationApp ptr obj - classificationptr
			Inherits Lifecyclehandler, optionshandler, and command recipient
		Register service options handler with classification ptr
		Register service lifecycle handler with classification ptr
		Register CommandRecipient with classification ptr
		Initialize ClassificationApp itself
			sets up internal readers and writers
			set up configuration options
		Initialize service
			set activity mode
			*SIMREADER
				registerrecipient (message handler)
					receive message
					message complete*
			All from config data
				build simreader for broadcast
				build simreader for commands
				build simwriter for replying to commands
		*TrackRepository::getInstance()
			Builds a trackrepository object and returns it*
		Trackrepository initialize
			From config data
				build simreader for tracks
					Trackreceiver (msg handler/sim recipient) init
					register trackreceiver as recipient
		Register trackerreader as a reader
		Sit in wait for shutdown loop
	*There are handlers in service that call notification handlers and message handlers*
		There is an internal shutdown
			shutsdown all internal readers
		And a service shutdown
			shutsdown track reader

Storing up in snapshots
	snapshots hold a varying number of received tracks
	snapshotintime list holds snapshotstates which holds system trackids: tracks
On stopreactive
	generating training data from snapshots 
		according to version
	writing to disk all training data for each version
		for a given scenario
			
PROBLEMS:
	snapshotintime is updated every 1 second
	what afsim updates would actually be