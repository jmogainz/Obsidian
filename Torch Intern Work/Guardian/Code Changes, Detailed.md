Extract Range and send in Message
    TESTS and other calls (ignore and create another constructor):
        TrackRepositoryTests.cpp (line 47, 67, 354)
        DISTrackParserTests.cpp (line 169)
        DISTrackParser.cpp (line 85)
    BuildMessage alter
        extract from track [range] [signal_props] [covariance] [range rate] [bearing] [sig to noise]
    SensorTrackMessage alter
        constructors (.h and .cpp) [range] [covariance] [range rate] [bearing] [sig to noise]
        decoder [range] [covariance] [range rate] [bearing] [sig to noise]
        encoder [range] [covariance] [range rate] [bearing] [sig to noise]
        getter [range] [covariance] [range rate] [bearing] [sig to noise]
        type initialization [range] [signal_props] [covariance] [range rate] [bearing] [sig to noise]
        priv var [range] [signal_props] [covariance] [range rate] [bearing] [sig to noise]
        calc size function [range] [covariance] [range rate] [bearing] [sig to noise]
        to_string [range] 
SystemTrack
        constructors (.h and .cpp) [range] [covariance] [range rate] [bearing] [sig to noise]
        toString debug function [range] 
        priv var [range] [covariance] [range rate] [bearing] [sig to noise]
        getter [range] [covariance] [range rate] [bearing] [sig to noise]
        setter [range] [covariance] [range rate] [bearing] [sig to noise]
Unpack Range from Message
    TrackService handlesourcetrackmess. alter
        constructor calls (2) [range] [covariance] [range rate] [bearing] [sig to noise]
        set call [range] [covariance] [range rate] [bearing] [sig to noise]
    sendTrackReport
        trackReportmessage
            constructor(track)(cpp) [range] [covariance] [range rate] [bearing] [sig to noise]
            constructor(.h and .cpp) [range] [covariance] [range rate] [bearing] [sig to noise]
            priv var [range] [covariance] [range rate] [bearing] [sig to noise]
			type init [range] [covariance] [range rate] [bearing] [sig to noise]
            to_string [range]
            calculate_size [range, maybe] [covariance] [range rate] [bearing] [sig to noise]
            min message size [range, maybe] [covariance] [range rate] [bearing] [sig to noise]
            getter [range] [covariance] [range rate] [bearing] [sig to noise]
        trackreportmessageparser
            createmessagefromtrackreportmessage [range] [covariance] [range rate] [bearing] [sig to noise]
            parsetotrackreportmessage [range] [covariance] [range rate] [bearing] [sig to noise]
				constructor call [range] [covariance] [range rate] [bearing] [sig to noise]
        trackreporttrackparser 
            parseTotrack [range] [covariance] [range rate] [bearing] [sig to noise]
	BUILD NOW
	AIMLClassificationAppLabeledFeatureSet
        ClassificationFeatures [range] [covariance] [range rate] [bearing] [sig to noise]
    AIMLClassificationAppLabeledFeatureCreator
        extractTrackRangeFeature (.h and .cpp) [range] [covariance] [range rate] [bearing] [sig to noise]
        generateSample (.h and .cpp) [range] [covariance] [range rate] [bearing] [sig to noise]
        generateTrainingdata [range] [covariance] [range rate] [bearing] [sig to noise]
	ClassificationApp
		handleSystemTrackRecipient [range] [covariance] [range rate] [bearing] [sig to noise]
    labeledfeaturecreator.savetodisk
        all versions [range] [covariance] [range rate] [bearing] [sig to noise]
        alter savetodisk <only filter necessary>

Temporary Changes to Prioritization:
    None, thanks to extra constructor on system track
        Multiple apps use system track
    None, thanks to extra constructor on track report
        Multiple apps use track reports

LOGGING CHANGES:
    cout in Sensorobserver afsim
    logger in Trackservice (two places)
    logger in labeledfeaturecreator (two places)

Quick test changes:
    Altered to reference in constructor
    Altered to reference in setter
    changed to double_t in extracter

Fix:
    Green under a systemtrack constuctor [fixed]
        initialized to default value

Recompile Toolbox
Recompile afsim
Add reports_range to main platform file
Recompile Guardian