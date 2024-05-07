Inherits from WsfRadarSensor
- Has perfect track quality?
Has specialiized RadarBeam
- Has perfect beam
Has specialied Tracker
Has specialized RadarMode
- Ignores terrain
Create on add platforms
Call update whenever other platforms update or detection attempt occurred and terrainChecked & failedStatus
- NO CALLBACKS GO OFF - ALL ARE SILENCED

ShadowWsfRadarSensor
- [ ] Modify terrain Masking mode of receiver for mode 1, beam1, on creation (POSSIBLY FOR TRANSMITTER?)
- [ ] Modify Sensor, Modify Rcvr configurations, Mode configurations, Beam configurations
	- [ ] Respectively
		- [ ] Create public setters in shadow class
		- [ ] GetEM_Rcvr, public setters
		- [ ] GetModeEntry(index), ShadowRadarMode inherits from RadarMode, access to all protected attributes, create public setters
		- [ ] GetModeEntry(index).GetBeamEntry(index) = Beam created through process input

ALL sensor callbacks will go off
THEN
AdvanceTime

Use FramePlatformsUpdated to scan with shadow sensors


