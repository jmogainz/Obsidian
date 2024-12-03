@startuml datarecorder_class_model_diagram

header
    Predictive Analysis RL C++ Environments Library
    UML Version 2.4.1
endheader
skinparam headerFontSize 24
skinparam headerFontStyle bold

skinparam linetype ortho
skinparam ClassAttributeIconSize 0
skinparam DefaultFontSize 20

/' Packages '/

package torch::datarecorder {

	/' Classes '/

	class DataRecorder <<singleton>> ##[bold]red {
		{static} -instance_ : DataRecorder*
		-clock_ : std::shared_ptr<toolbox::timelib::ReadOnlySimClock>
		-recording_interval : toolbox::timelib::TimeUtil::MSecOfLogicalTime
		-registered_ : std::vector<std::unique_ptr<AbstractRecordable>>
		-state_mutex_ : std::mutex

		{static} +getInstanceIfInitialized() : DataRecorder&
		+addRecordable(obj : std::unique_ptr<AbstractRecordable>) : void
		+startRecording() : void
		+stopRecording() : void
		#DataRecorder()
		-recordAll() : void
	}

	class DataRecorder::Builder ##[bold]red {
		-recording_interval_ : int
		-registered_ : std::vector<std::unique_ptr<AbstractRecordable>>
		-clock_ : std::shared_ptr<toolbox::timelib::ReadOnlySimClock>

		+setWithYaml(file_path : const std::string&) : Builder&
		+setRecordingInterval(interval : toolbox::timelib::TimeUtil::MSecOfLogicalTime) : Builder&
		+setClock(clock : std::shared_ptr<toolbox::timelib::ReadOnlySimClock>) : Builder&
		+buildSingleton() : void
	}

	interface AbstractRecordable ##[bold]red {
		{abstract} +getRecordData() : MixedTypeMap::RecursiveMap
	}

	/' Relationships'/

	DataRecorder "1" o-- "0..*" AbstractRecordable #line:red
	DataRecorder::Builder ..> DataRecorder #line:red
}

together {
package tracktools::tracks {

	/' Classes '/

	class TrackRepositoryManager <<singleton>> {
		#repositories_ : std::map<std::string, std::shared_ptr<TrackRepository>>
		#trackParser_ : std::unique_ptr<TrackParser>

		#TrackRepositoryManager()

		+{static} getInstance() : TrackRepositoryManager&
		+initialize(const std::shared_ptr<toolbox::config::StructuredData>& configData, const toolbox::timelib::ReadOnlySimClock& clock) : bool
		+getRepository(const std::string& vehicleId) : std::shared_ptr<TrackRepository>
		+shutdown() : void
		#purgeTimeLimitAllRepositories() : void
		#receiveMessage(const toolbox::messages::Message& message) : void
		#resetAllRepositories() : void
	}

	class PositionRepositoryManager <<singleton>> {
		#repositories_ : std::map<std::string, std::shared_ptr<PositionRepository>>

		#PositionRepositoryManager()

		+{static} getInstance() : PositionRepositoryManager&
		+initialize(const std::shared_ptr<toolbox::config::StructuredData>& configData, const toolbox::timelib::ReadOnlySimClock& clock) : bool
		+getRepository(const std::string& vehicleId) : std::shared_ptr<PositionRepository>
		+shutdown() : void
		#purgeTimeLimitAllRepositories() : void
		#receiveMessage(const toolbox::messages::Message& message) : void
		#resetAllRepositories() : void
	}

}

together {

package torch::machinelearning::reinforcementlearning::clairvoyanttrackrepository {

	/' Classes '/

	class ClairvoyantTrackRepositoryManager <<singleton>> {
		-CLAIRVOYANT_TRACK_REPOSITORY_MANAGER_NAME : const std::string

		#ClairvoyantTrackRepositoryManager()

		+{static} getInstance() : ClairvoyantTrackRepositoryManager&
		+initialize(configData : const std::shared_ptr<toolbox::config::StructuredData>&, clock : const toolbox::timelib::ReadOnlySimClock&) : bool
	}

	class ClairvoyantTrackRepositoryManager::RecordingBridge ##[bold]red {
		- parent_ : ClairvoyantTrackRepositoryManager&
		
		+RecordingBridge(parent : ClairvoyantTrackRepositoryManager&)
		+getRecordData() : MixedTypeMap::RecursiveMap
	}

	/' Relationships '/

	ClairvoyantTrackRepositoryManager -up-|> TrackRepositoryManager
	ClairvoyantTrackRepositoryManager ..> ClairvoyantTrackRepositoryManager::RecordingBridge #line:red
	ClairvoyantTrackRepositoryManager ..> DataRecorder #line:red
	ClairvoyantTrackRepositoryManager::RecordingBridge ..|> AbstractRecordable #line:red
}

}

together {

package torch::machinelearning::reinforcementlearning::sensorlimitsrepository {

	/' Classes '/

	class SensorLimitsRepositoryManager <<singleton>> {
		#SensorLimitsRepositoryManager()
		+~SensorLimitsRepositoryManager()
		+{static} getInstance() : SensorLimitsRepositoryManager&
		+hasRepositoryFor(vehicleId : const std::string&) : bool
		+initialize(configData : const std::shared_ptr<toolbox::config::StructuredData>&, clock : const toolbox::timelib::ReadOnlySimClock&) : bool
		+getRepository(vehicleId : const std::string&) : std::shared_ptr<SensorLimitsRepository>
		+forceUpdateRepositoriesFromMessage(cmd : const std::unique_ptr<toolbox::messages::CommandMessage>&) : void
		+shutdown() : void
		
		#repositories_ : std::map<std::string, std::shared_ptr<SensorLimitsRepository>>
		#checkAllRepositoriesForStaleData_StaleDataLocked() : void
		#purgeTimeLimitAllRepositories_NotLocked() : void
		#receiveMessage(message : const toolbox::messages::Message&) : void
		#resetAllRepositories_Locked() : void
	}

	class SensorLimitsRepositoryManager::RecordingBridge ##[bold]red {
		- parent_ : SensorLimitsRepositoryManager&
		
		+RecordingBridge(parent : SensorLimitsRepositoryManager&)
		+getRecordData() : MixedTypeMap::RecursiveMap
	}

	/' Relationships '/

	SensorLimitsRepositoryManager ..> SensorLimitsRepositoryManager::RecordingBridge #line:red
	SensorLimitsRepositoryManager ..> DataRecorder #line:red
	SensorLimitsRepositoryManager::RecordingBridge ..|> AbstractRecordable #line:red
}

}

package torch::machinelearning::reinforcementlearning::trackrepository {

	/' Classes '/

	class TrackRepositoryManagerDRAdapter ##[bold]red {
		{static} +getInstance() : TrackRepositoryManagerDRAdapter&
		<<override>> +initialize(const std::shared_ptr<toolbox::config::StructuredData>& configData, const toolbox::timelib::ReadOnlySimClock& clock) : bool
		#TrackRepositoryManagerDRAdapter()
	}

	class TrackRepositoryManagerDRAdapter::RecordingBridge ##[bold]red {
		- parent_ : TrackRepositoryManagerDRAdapter&

		+RecordingBridge(parent : TrackRepositoryManagerDRAdapter&)
		+getRecordData() : MixedTypeMap::RecursiveMap
	}

	/' Relationships '/

	TrackRepositoryManagerDRAdapter -down-|> TrackRepositoryManager #line:red
	TrackRepositoryManagerDRAdapter ..> TrackRepositoryManagerDRAdapter::RecordingBridge #line:red
	TrackRepositoryManagerDRAdapter ..> DataRecorder #line:red
	TrackRepositoryManagerDRAdapter::RecordingBridge ..|> AbstractRecordable #line:red
	
}

package torch::machinelearning::reinforcementlearning::positionrepository {

	/' Classes '/

	class PositionRepositoryManagerDRAdapter ##[bold]red {
		{static} +getInstance() : PositionRepositoryManagerDRAdapter&
		<<override>> +initialize(const std::shared_ptr<toolbox::config::StructuredData>& configData, const toolbox::timelib::ReadOnlySimClock& clock) : bool
		#PositionRepositoryManagerDRAdapter()
	}

	class PositionRepositoryManagerDRAdapter::RecordingBridge ##[bold]red {
		- parent_ : PositionRepositoryManagerDRAdapter&

		+RecordingBridge(parent : PositionRepositoryManagerDRAdapter&)
		+getRecordData() : MixedTypeMap::RecursiveMap
	}

	/' Relationships '/

	PositionRepositoryManagerDRAdapter -down-|> PositionRepositoryManager #line:red
	PositionRepositoryManagerDRAdapter ..> PositionRepositoryManagerDRAdapter::RecordingBridge #line:red
	PositionRepositoryManagerDRAdapter ..> DataRecorder #line:red
	PositionRepositoryManagerDRAdapter::RecordingBridge ..|> AbstractRecordable #line:red

}

package torch::machinelearning::reinforcementlearning::dataextraction {
	
	/' Classes '/

	class DataExtractionRepository {
		-DataExtractionRepository()
		+{static} getInstance() : DataExtractionRepository&
		+getPlatformHistoryFromSnapshot(const std::string& platform_id, std::shared_ptr<std::vector<std::shared_ptr<tracktools::tracks::PositionData>>>& history) : bool
		+getSensedTrackHistoryFromSnapshot(const std::string& sensor_id, const std::string& track_id, std::shared_ptr<std::vector<std::shared_ptr<tracktools::tracks::SystemTrack>>>& history) : bool
		+getSensedTrackIdsFromSnapshot(const std::string& sensor_id, std::shared_ptr<std::vector<std::string>>& ids) : bool
		+getSimRelativeTimeFromSnapshot(toolbox::timelib::TimeUtil::MSecOfLogicalTime& time) : bool
		+getTruthTrackHistoryFromSnapshot(const std::string& sensor_id, const std::string& track_id, std::shared_ptr<std::vector<std::shared_ptr<tracktools::tracks::SystemTrack>>>& history) : bool
		+getTruthTrackIdsFromSnapshot(const std::string& sensor_id, std::shared_ptr<std::vector<std::string>>& ids) : bool
		-lockedGetPositionRepoFromPlatform(const std::string& platform_id, std::shared_ptr<tracktools::tracks::PositionRepository>& repo) : bool
		-lockedGetTrackRepoFromClairvoyantSensor(const std::string& sensor_id, std::shared_ptr<tracktools::tracks::TrackRepository>& repo) : bool
		-lockedGetTrackRepoFromSensor(const std::string& sensor_id, std::shared_ptr<tracktools::tracks::TrackRepository>& repo) : bool
		+retrieveObserver(const std::string& observer_id, std::shared_ptr<Observer>& observer) : bool
		-repository_state_mutex_ : std::mutex
		-snapshot_ : std::shared_ptr<SnapshotData>
		+getAllPlatformIdsFromSnapshot() : std::shared_ptr<std::vector<std::string>>
		+getAllSensorIdsFromSnapshot() : std::shared_ptr<std::vector<std::string>>
		-logger_ : std::shared_ptr<toolbox::logger::Logger>
		-sim_clock_ : std::unique_ptr<toolbox::timelib::ReadOnlySimClock>
		-registered_observers_wrappers_ : std::unordered_map<std::type_index, std::unique_ptr<IRegisteredObserversWrapper>>
		-ensureSnapshotInitialized() : void
		-lockedTakeClairvoyantSensorTrackHistorySnapshot() : void
		-lockedTakePlatformHistorySnapshot() : void
		-lockedTakeRelativeTimeSnapshot() : void
		-lockedTakeSensorTrackHistorySnapshot() : void
		-lockedTakeSnapshot() : void
		+registerObserver(const std::string& observer_id, std::shared_ptr<Observer> observer) : void
		+removeSimClock() : void
		+resetRepository() : void
		+resetRepositoryData() : void
		+setSimClock(const toolbox::timelib::ReadOnlySimClock& sim_clock) : void
		+takeSnapshot() : void
	}

	class DataExtractionRepository::RecordingBridge ##[bold]red {
		- parent_ : DataExtractionRepository&

		+RecordingBridge(parent : DataExtractionRepository&)
		+getRecordData() : MixedTypeMap::RecursiveMap
	}

	/' Relationships '/

	DataExtractionRepository ..> DataExtractionRepository::RecordingBridge #line:red
	DataExtractionRepository ..> DataRecorder #line:red
	DataExtractionRepository::RecordingBridge ..|> AbstractRecordable #line:red
}

}

package torch::machinelearning::reinforcementlearning::environment {

	/' Classes '/

	class Environment {
		+dataObserverLifecycleHandler_ : std::shared_ptr<DataObserverLifecycleHandler>

		-episode_ : size_t
		-timestep_ : size_t

		-waitForEndBarrier_ : std::shared_ptr<std::atomic_bool>
		-waitForFinalFrameBarrier_ : std::shared_ptr<std::atomic_bool>
		-waitForFrameAdvanceBarrier_ : std::shared_ptr<std::atomic_bool>
		-waitForSensorLimitsBarrier_ : std::shared_ptr<std::atomic_bool>
		-waitForStartBarrier_ : std::shared_ptr<std::atomic_bool>
		-shutdownEnvironment_ : std::shared_ptr<std::atomic_bool>

		-scenarioEndBarrier_ : std::shared_ptr<torch::threadUtilities::Barrier>
		-scenarioFinalFrameBarrier_ : std::shared_ptr<torch::threadUtilities::Barrier>
		-scenarioFrameAdvanceBarrier_ : std::shared_ptr<torch::threadUtilities::Barrier>
		-scenarioSensorLimitsBarrier_ : std::shared_ptr<torch::threadUtilities::Barrier>
		-scenarioStartBarrier_ : std::shared_ptr<torch::threadUtilities::Barrier>

		-config_ : std::shared_ptr<toolbox::config::StructuredData>
		-logger_ : std::shared_ptr<toolbox::logger::Logger>

		-action_handler_reader_ : std::shared_ptr<toolbox::messages::SimReader>
		-action_handler_writer_ : std::shared_ptr<toolbox::messages::SimWriter>
		-track_service_writer_ : std::shared_ptr<toolbox::messages::SimWriter>

		-temporalFluxCapacitor_ : std::unique_ptr<TemporalFluxCapacitor>

		+Environment()
		
		+ <<override>> reset(int seedValue, const torch::mapUtilities::MixedTypeMap::RecursiveVariant& options) : IEnvironment::ResetResults
		+ <<override>> step(const torch::mapUtilities::MixedTypeMap::RecursiveVariant& myAction) : IEnvironment::StepResults
		+ shutdown() : void
		+isEnvironmentShutdown() : bool
		+initialize(uint16_t environment_id, const std::string& initialAppName, const std::string& initialAppInstance, const std::filesystem::path& run_path,\n\t\t const std::filesystem::path& service_allocator_config_path, const std::string& service_allocator_config_file_name,\n\t\t const std::filesystem::path& afsim_scenarios_path, const std::string& afsim_comms_config_template_file_name,\n\t\t const std::filesystem::path& action_observation_space_config_path, const std::string& action_observation_space_config_file_name,\n\t\t const std::filesystem::path& install_library_path, const int argc, const char** argv) : bool

		-initializeEnvironmentSingletons(const std::shared_ptr<toolbox::service::Service>& service) : void
		-initializeForVectorization(uint16_t environment_id, const std::filesystem::path& service_allocator_config_file_path, const std::filesystem::path& afsim_scenarios_path,\n\t\t const std::string& afsim_comms_config_template_file_name, const std::filesystem::path& run_path, const int argc, const char** argv) : void
		-initializeSteppingAlgorithm(const std::filesystem::path& environment_step_config_file_path, const std::filesystem::path& install_library_path, const std::shared_ptr<toolbox::service::Service>& service) : void
		-configureStepSynchronization(bool stepFullScenario) : void
		-setupActionHandler(const std::filesystem::path& install_library_path, const std::string& action_library_name, const std::shared_ptr<toolbox::service::Service>& service,\n\t\t const std::shared_ptr<torch::machinelearning::reinforcementlearning::space::ISpace>& action_space) : void
		-setupObservationHandler(const std::filesystem::path& install_library_path, const std::string& observation_library_name, const std::shared_ptr<torch::machinelearning::reinforcementlearning::space::ISpace>& observation_space,\n\t\t const std::optional<std::shared_ptr<torch::machinelearning::reinforcementlearning::instructions::Instructions>>& observation_instructions) : void
		-setupRewardHandler(const std::filesystem::path& install_library_path, const std::string& reward_library_name, const std::shared_ptr<torch::machinelearning::reinforcementlearning::reward::RewardCommon>& reward_common,\n\t\t const std::optional<std::shared_ptr<torch::machinelearning::reinforcementlearning::instructions::Instructions>>& reward_instructions,\n\t\t const std::optional<std::shared_ptr<torch::mapUtilities::MixedTypeMap::RecursiveMap>>& reward_info_node) : void
		-startTemporalFluxCapacitor(const std::string& initialAppName, const std::string& initialAppInstance, const int argc, const char** argv) : void

		-sendSensorLimitsRequestMessage(const std::shared_ptr<toolbox::service::Service>& service) : void

		-closeEnvironmentReadersAndWriters() : void
		-setupAReader(bool& isInitialized, const std::string& configName, const std::shared_ptr<toolbox::config::StructuredData>& environmentConfig) : std::shared_ptr<toolbox::messages::SimReader>
		-setupAWriter(bool& isInitialized, const std::string& configName, const std::shared_ptr<toolbox::config::StructuredData>& environmentConfig) : std::shared_ptr<toolbox::messages::SimWriter>
	}

	class Environment::DataObserverLifecycleHandler {
		+DataObserverLifecycleHandler(Environment& theParent)
		-parent_ : Environment&
		+resetState() : void
		+shutdown() : void
		+startListening() : void
		+stopListening() : void
	}

	class Environment::RecordingBridge ##[bold]red {
		- parent_ : Environment&

		+RecordingBridge(parent : Environment&)
		+getRecordData() : MixedTypeMap::RecursiveMap
	}

	class EnvironmentParser {
		+{static} getEnvironmentStepConfig(const std::filesystem::path& config_Path) : std::shared_ptr<EnvironmentStepConfig>
		+{static} parseYaml(const std::filesystem::path& config_path) : std::shared_ptr<YAML::Node>
		-{static} join(const std::vector<std::string>& elements, const std::string& delimiter) : std::string
		+{static} validateYaml(const std::shared_ptr<YAML::Node>& values) : void
	}

	interface IDataObserverLifecycle <<interface>> {
		+~IDataObserverLifecycle()
		+{abstract} resetState() : void
		+{abstract} shutdown() : void
		+{abstract} startListening() : void
		+{abstract} stopListening() : void
	}

	abstract class IEnvironment <<abstract>> {
		#step_results_ : StepResults
		#seed : int
		#action_handler_ : std::shared_ptr<action::IActionHandler>
		#action_loader_ : std::shared_ptr<genericLibraryLoader::GenericLibraryLoader>
		#observation_loader_ : std::shared_ptr<genericLibraryLoader::GenericLibraryLoader>
		#reward_loader_ : std::shared_ptr<genericLibraryLoader::GenericLibraryLoader>
		#observation_handler_ : std::shared_ptr<observation::IObservationHandler>
		#reward_handler_ : std::shared_ptr<reward::IRewardHandler>

		+ IEnvironment()
		+{abstract} reset(int seedValue, const torch::mapUtilities::MixedTypeMap::RecursiveVariant& options) : ResetResults
		+{abstract} step(const torch::mapUtilities::MixedTypeMap::RecursiveVariant& myAction) : StepResults
		+{abstract} shutdown() : void
		+getLatestReward() : double
		+getSeed() : int
		+getActionSpace() : std::shared_ptr<space::ISpace>
		+getObservationSpace() : std::shared_ptr<space::ISpace>
		+getLatestObservation() : torch::mapUtilities::MixedTypeMap::RecursiveVariant&
		+setSeed(int value) : void
	}

	class TemporalFluxCapacitor {
		-service_ : std::shared_ptr<toolbox::service::Service>
		-lifecycleHandler_ : std::shared_ptr<LifecycleHandler>
		-optionsHandler_ : std::shared_ptr<OptionsHandler>
		-timestepHandler_ : std::shared_ptr<TimestepHandler>
		-replyHandler_ : std::shared_ptr<ReplyRecipient>
		-dataObserverLifecycleHandler_ : std::shared_ptr<IDataObserverLifecycle>

		-waitForEndBarrier_ : std::shared_ptr<std::atomic_bool>
		-waitForFinalFrameBarrier_ : std::shared_ptr<std::atomic_bool>
		-waitForFrameAdvanceBarrier_ : std::shared_ptr<std::atomic_bool>
		-waitForSensorLimitsBarrier_ : std::shared_ptr<std::atomic_bool>
		-waitForStartBarrier_ : std::shared_ptr<std::atomic_bool>
		-shutdownEnvironment_ : std::shared_ptr<std::atomic_bool>

		-scenarioEndBarrier_ : std::shared_ptr<torch::threadUtilities::Barrier>
		-scenarioFinalFrameBarrier_ : std::shared_ptr<torch::threadUtilities::Barrier>
		-scenarioFrameAdvanceBarrier_ : std::shared_ptr<torch::threadUtilities::Barrier>
		-scenarioSensorLimitsBarrier_ : std::shared_ptr<torch::threadUtilities::Barrier>
		-scenarioStartBarrier_ : std::shared_ptr<torch::threadUtilities::Barrier>

		-logger_ : std::shared_ptr<toolbox::logger::Logger>

		+TemporalFluxCapacitor(const std::string& initialAppName, const std::string& initialAppInstance, const int argc, const char** argv)
		+initialize(std::shared_ptr<toolbox::config::StructuredData>& config) : bool
		+getReplyHandler() : std::shared_ptr<ReplyRecipient> {query}
		+getService() : std::shared_ptr<toolbox::service::Service>
		+shutdown() : void

		+registerDataObserverLifecycleHandler(const std::shared_ptr<IDataObserverLifecycle>& newHandler) : void
		+registerScenarioEndBarrier(const std::shared_ptr<torch::threadUtilities::Barrier>& newBarrier, const std::shared_ptr<std::atomic_bool>& newWaitForBarrier) : void
		+registerScenarioFinalFrameBarrier(const std::shared_ptr<torch::threadUtilities::Barrier>& newBarrier, const std::shared_ptr<std::atomic_bool>& newWaitForBarrier) : void
		+registerScenarioFrameAdvanceBarrier(const std::shared_ptr<torch::threadUtilities::Barrier>& newBarrier, const std::shared_ptr<std::atomic_bool>& newWaitForBarrier) : void
		+registerScenarioSensorLimitsBarrier(const std::shared_ptr<torch::threadUtilities::Barrier>& newBarrier, const std::shared_ptr<std::atomic_bool>& newWaitForBarrier) : void
		+registerScenarioStartBarrier(const std::shared_ptr<torch::threadUtilities::Barrier>& newBarrier, const std::shared_ptr<std::atomic_bool>& newWaitForBarrier) : void
		+registerShutdownEnvironmentFlag(const std::shared_ptr<std::atomic_bool>& shutdownEnvironmentFlag) : void
	}

	together {
		class TemporalFluxCapacitor::LifecycleHandler {
			-parent_ : TemporalFluxCapacitor&

			+LifecycleHandler(TemporalFluxCapacitor& theParent)
			+shutdown(toolbox::service::Service& service) : void
			+startProactiveProcessing(toolbox::service::Service& service) : void
			+startReactiveProcessing(toolbox::service::Service& service) : void
			+stopProactiveProcessing(toolbox::service::Service& service) : void
			+stopReactiveProcessing(toolbox::service::Service& service) : void
		}

		class TemporalFluxCapacitor::OptionsHandler {
			-parent_ : TemporalFluxCapacitor&

			+OptionsHandler(TemporalFluxCapacitor& theParent)
			+optionChanged(toolbox::service::Service& service, std::shared_ptr<toolbox::service::ServiceOption>& option) : void
		}

		class TemporalFluxCapacitor::ReplyRecipient {
			-parent_ : TemporalFluxCapacitor&

			+ReplyRecipient(TemporalFluxCapacitor& theParent)
			+receiveMessageSensorLimitsConditionVar_ : std::condition_variable
			+readerComplete(const toolbox::messages::SimReader& reader) : void
			+receiveMessage(const toolbox::messages::Message& message) : void
		}

		class TemporalFluxCapacitor::TimestepHandler {
			-parent_ : TemporalFluxCapacitor&

			+TimestepHandler(TemporalFluxCapacitor& theParent)
			+advanceLogicalTime(toolbox::service::Service& service, toolbox::timelib::TimeUtil::MSecOfLogicalTime timeStep) : void
			+prepareLogicalTime(toolbox::service::Service& service) : void
		}
	}

	class EnvironmentStepConfig {
		+observation_instructions : std::optional<std::shared_ptr<torch::machinelearning::reinforcementlearning::instructions::Instructions>>
		+reward_instructions : std::optional<std::shared_ptr<torch::machinelearning::reinforcementlearning::instructions::Instructions>>
		+reward_space_params : std::optional<std::shared_ptr<torch::machinelearning::reinforcementlearning::space::SpaceParams>>
		+reward_info_node : std::optional<std::shared_ptr<torch::mapUtilities::MixedTypeMap::RecursiveMap>>
		+reward_common : std::shared_ptr<torch::machinelearning::reinforcementlearning::reward::RewardCommon>
		+action_space : std::shared_ptr<torch::machinelearning::reinforcementlearning::space::ISpace>
		+observation_space : std::shared_ptr<torch::machinelearning::reinforcementlearning::space::ISpace>
		+observation_space_params : std::shared_ptr<torch::machinelearning::reinforcementlearning::space::SpaceParams>
		+action_library : std::string
		+observation_library : std::string
		+reward_library : std::string
	}

	together {
		class IEnvironment::ResetResults {
			+info : torch::mapUtilities::MixedTypeMap::RecursiveVariant
			+observation : torch::mapUtilities::MixedTypeMap::RecursiveVariant
		}

		class IEnvironment::StepResults {
			+terminated : bool
			+truncated : bool
			+reward : double
			+info : torch::mapUtilities::MixedTypeMap::RecursiveMap
			+observation : torch::mapUtilities::MixedTypeMap::RecursiveVariant
		}
	}

	/' Inheritance relationships '/

	IEnvironment <|-- Environment

	IDataObserverLifecycle <|.. Environment::DataObserverLifecycleHandler

	/' Composition relationships '/

	Environment *-- TemporalFluxCapacitor
	Environment *-- Environment::DataObserverLifecycleHandler

	TemporalFluxCapacitor *-- TemporalFluxCapacitor::LifecycleHandler
	TemporalFluxCapacitor *-- TemporalFluxCapacitor::OptionsHandler
	TemporalFluxCapacitor *-- TemporalFluxCapacitor::ReplyRecipient
	TemporalFluxCapacitor *-- TemporalFluxCapacitor::TimestepHandler

	/' Aggregation relationships '/

	TemporalFluxCapacitor o-right- IDataObserverLifecycle

	/' Dependency relationships '/

	Environment .up.> IEnvironment::StepResults
	Environment .up.> IEnvironment::ResetResults
	Environment .left.> EnvironmentParser
	Environment .left.> EnvironmentStepConfig
	Environment ..> DataExtractionRepository
	Environment ..> TrackRepositoryManagerDRAdapter #line:red
	Environment ..> PositionRepositoryManagerDRAdapter #line:red
	Environment ..> ClairvoyantTrackRepositoryManager
	Environment ..> SensorLimitsRepositoryManager
	Environment ..> DataRecorder::Builder #line:red
	Environment .right.> Environment::RecordingBridge #line:red

	Environment::RecordingBridge ..|> AbstractRecordable #line:red

}

@enduml

