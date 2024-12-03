[DEBUG] distools_time_controller Receive Message: AdvanceTime;19935
[DEBUG] distools_time_controller ExternalTime: 19935
[DEBUG] distools_time_controller AFSIM released.
[DEBUG] distools_time_controller Preparing advance time.
[DEBUG] distools_time_controller AFSIM attempting to advance to: 19935
[DEBUG] distools_time_controller Write Message: CompleteAdvanceTime;19935
[DEBUG] distools_time_controller AFSIM caught up to external. Notifying external of timestep complete.
[DEBUG] distools_time_controller Advancing - AFSIM Time: 19935
[2024-11-15 12:36:26.917] {afsimbridge_ServiceGroup_1} [debug] Received data from Time Control Plugin: CompleteAdvanceTime;19935

[2024-11-15 12:36:26.917] {afsimbridge_ServiceGroup_1} [debug] Time Control Plugin sent CompleteAdvanceTime: 19935

[2024-11-15 12:36:26.917] {afsimbridge_ServiceGroup_1} [debug] TimeMessageHandler notified parent.
[2024-11-15 12:36:26.917] {afsimbridge_ServiceGroup_1} [debug] Completing advance time to 19935
[2024-11-15 12:36:26.917] {afsimbridge_ServiceGroup_1} [debug] Service reply-writing 'AdvanceLogicalTime' using (UDP:127.0.0.1:13631)
[2024-11-15 12:36:26.918] {afsimbridge_ServiceGroup_1} [debug] Service advancing logical time
[2024-11-15 12:36:26.918] {afsimbridge_ServiceGroup_1} [debug] Clock now at 2024-11-15 12:36:44.880, Relative=19980
[2024-11-15 12:36:26.918] {afsimbridge_ServiceGroup_1} [debug] Publishing: AdvanceTime
[DEBUG] distools_time_controller Receive Message: AdvanceTime;19980
[DEBUG] distools_time_controller ExternalTime: 19980
[DEBUG] distools_time_controller AFSIM released.
[DEBUG] distools_time_controller Preparing advance time.
[DEBUG] distools_time_controller AFSIM attempting to advance to: 19980
[DEBUG] distools_time_controller Write Message: CompleteAdvanceTime;19980
[DEBUG] distools_time_controller AFSIM caught up to external. Notifying external of timestep complete.
[DEBUG] distools_time_controller Advancing - AFSIM Time: 19980
[2024-11-15 12:36:26.921] {afsimbridge_ServiceGroup_1} [debug] Received data from Time Control Plugin: CompleteAdvanceTime;19980

[2024-11-15 12:36:26.921] {afsimbridge_ServiceGroup_1} [debug] Time Control Plugin sent CompleteAdvanceTime: 19980

[2024-11-15 12:36:26.921] {afsimbridge_ServiceGroup_1} [debug] TimeMessageHandler notified parent.
[2024-11-15 12:36:26.921] {afsimbridge_ServiceGroup_1} [debug] Completing advance time to 19980
[2024-11-15 12:36:26.921] {afsimbridge_ServiceGroup_1} [debug] Service reply-writing 'AdvanceLogicalTime' using (UDP:127.0.0.1:13631)
[2024-11-15 12:36:26.924] {afsimbridge_ServiceGroup_1} [debug] Service stopping proactive processing
[2024-11-15 12:36:26.924] {afsimbridge_ServiceGroup_1} [debug] Pausing clock at 2024-11-15 12:36:44.880, Relative=19980
[2024-11-15 12:36:26.924] {afsimbridge_ServiceGroup_1} [info] Sending Pause (Recess) PDU to AFSIM.
[2024-11-15 12:36:26.924] {afsimbridge_ServiceGroup_1} [debug] AFSIM state set to 'Paused'
[2024-11-15 12:36:26.924] {afsimbridge_ServiceGroup_1} [debug] Service reply-writing 'StoppedProactiveProcessing' using (UDP:127.0.0.1:13631)
[2024-11-15 12:36:26.925] {afsimbridge_ServiceGroup_1} [debug] Service stopping reactive processing
[2024-11-15 12:36:26.925] {afsimbridge_ServiceGroup_1} [debug] Stopping clock at 2024-11-15 12:36:44.880, Relative=19980
[2024-11-15 12:36:26.925] {afsimbridge_ServiceGroup_1} [debug] AFSIM state set to 'Stopped'
[2024-11-15 12:36:26.925] {afsimbridge_ServiceGroup_1} [info] Reactive Stopped.  DIS bridging disabled.
[2024-11-15 12:36:26.925] {afsimbridge_ServiceGroup_1} [debug] Service reply-writing 'StoppedReactiveProcessing' using (UDP:127.0.0.1:13631)
[DEBUG] distools_time_controller Preparing advance time.
[DEBUG] distools_time_controller AFSIM attempting to advance to: 20025
[DEBUG] distools_time_controller Waiting for external clock.
[2024-11-15 12:36:26.944] {afsimbridge_ServiceGroup_1} [debug] Service received 'SetOptions'
[2024-11-15 12:36:26.944] {afsimbridge_ServiceGroup_1} [debug] Service setting option ScenarioFilename
[2024-11-15 12:36:26.944] {afsimbridge_ServiceGroup_1} [debug] Service reply-writing 'OptionList' using (UDP:127.0.0.1:13631)
[2024-11-15 12:36:26.976] {afsimbridge_ServiceGroup_1} [debug] Service starting logical time processing.  RunName='ServiceGroup_1-0004', Playback=0
[2024-11-15 12:36:26.976] {afsimbridge_ServiceGroup_1} [debug] Starting clock at 2024-11-15 12:36:26.975, Relative=0
[2024-11-15 12:36:26.976] {afsimbridge_ServiceGroup_1} [debug] Current  clock at 2024-11-15 12:36:26.975, Relative=0
[2024-11-15 12:36:26.976] {afsimbridge_ServiceGroup_1} [info] Shutting down previous AFSIM instance due to option change.
[2024-11-15 12:36:26.976] {afsimbridge_ServiceGroup_1} [info] Attempting to kill child AFSIM process.
[2024-11-15 12:36:26.976] {afsimbridge_ServiceGroup_1} [debug] Publishing: TerminateTimeControlPlugin
[DEBUG] distools_time_controller Receive Message: TerminateTimeControlPlugin;1
[DEBUG] distools_time_controller Write Message: TerminatedTimeControlPlugin;1
[2024-11-15 12:36:26.977] {afsimbridge_ServiceGroup_1} [debug] Received data from Time Control Plugin: TerminatedTimeControlPlugin;1

[2024-11-15 12:36:26.977] {afsimbridge_ServiceGroup_1} [info] AFSIM time control plugin terminated successfully :true
[2024-11-15 12:36:26.993] {afsimbridge_ServiceGroup_1} [debug] AFSIM successfully terminated.
[2024-11-15 12:36:26.993] {afsimbridge_ServiceGroup_1} [debug] PID return from waitpid: -1
[2024-11-15 12:36:26.993] {afsimbridge_ServiceGroup_1} [info] AFSIM process completed execution.
[2024-11-15 12:36:26.993] {afsimbridge_ServiceGroup_1} [debug] AFSIM state set to 'Stopped'
[2024-11-15 12:36:26.993] {afsimbridge_ServiceGroup_1} [debug] AFSIM state set to 'Stopped'
[2024-11-15 12:36:26.993] {afsimbridge_ServiceGroup_1} [info] Attempting to start AFSIM process.
[2024-11-15 12:36:26.994] {afsimbridge_ServiceGroup_1} [info] Child process created to execute AFSIM with PID: 20135
[2024-11-15 12:36:26.994] {afsimbridge_ServiceGroup_1} [debug] AFSIM process started success
[2024-11-15 12:36:26.994] {afsimbridge_ServiceGroup_1} [debug] AFSIM state set to 'Running'
[2024-11-15 12:36:26.994] {afsimbridge_ServiceGroup_1} [info] Preparing Logical Time.  DIS bridging enabled.
[2024-11-15 12:36:26.994] {afsimbridge_ServiceGroup_1} [debug] Publishing: EnablePlugin
[2024-11-15 12:36:26.994] {afsimbridge_ServiceGroup_1} [debug] Waiting in Prepare Logical Time for AFSIM process to start...
mission:
    Product Version: 2.2211.0-nongit 2024-11-15 (non-git)[0000000000]
    Internal Version: 2.2211.0
Plugin API version info:
    Version: 2.2211
    System ABI: x86_64-linux-gnu^gcc1104,debug,hwe,noglibc11abi
Plugins Loaded:
    libwsf_alternate_locations.so, libwsf_coverage.so, libwsf_iads_c2.so, libwsf_prompt.so, libwsf_scenario_analyzer.so, libwsf_scenario_analyzer_iads_c2.so, libclairvoyant_sensor_observer.so, libdistools_time_controller.so, libplatform_teleporter.so, libtracktools_sensor_observer.so, libwsf_air_combat.so, libwsf_annotation.so, libwsf_brawler.so, libwsf_fires.so, libwsf_fuzzy_logic.so, libwsf_gis.so, libwsf_multiresolution.so, libwsf_multiresolution_intrinsic.so, libwsf_multiresolution_metamodel.so, libwsf_oms_uci.so, libwsf_p6dof.so, libwsf_simdis.so, libwsf_six_dof.so, libwsf_sosm.so
Extensions Included: 
    sensor_plot_lib, wsf_advanced_behavior, wsf_cyber, wsf_grammar_check, wsf_l16, wsf_mil, wsf_mil_parser, wsf_mtt, wsf_nx, wsf_parser, wsf_ripr, wsf_space, wsf_weapon_server

Loading simulation input.
Loading simulation input complete.
    Elapsed Wall Clock Time: 0.03534
    Elapsed Processor Time : 0.0353
Initializing simulation.
***** ERROR: Unable to open event_pipe file.
    File: ./output/example_scenario_seed_2_env_ServiceGroup_1.aer
Activating DIS connection.
    T = 0
    Unicast: 127.0.0.1
    Sending Port: 12346
    Receiving Port: 12347
[DEBUG] distools_time_controller Set up UDP connection.
Success in Connection Status for the DisTools Time Control Plugin Network Connection
[DEBUG] distools_time_controller Write Message: PluginInitialized;1
[DEBUG] PlatformTeleporter: Success in Connection Status for the Platform Teleporter Plugin Network Connection
[2024-11-15 12:36:27.802] {afsimbridge_ServiceGroup_1} [debug] Received data from Time Control Plugin: PluginInitialized;1

[2024-11-15 12:36:27.803] {afsimbridge_ServiceGroup_1} [debug] Time Control Plugin initialized. EnablePlugin: 1
[2024-11-15 12:36:27.803] {afsimbridge_ServiceGroup_1} [debug] Publishing: EnablePlugin
TrackTools Sensor Observer has a valid config.
[DEBUG] distools_time_controller Receive Message: EnablePlugin;1
[DEBUG] distools_time_controller Write Message: EnablePlugin;1
[DEBUG] distools_time_controller ENABLED
[2024-11-15 12:36:27.803] {afsimbridge_ServiceGroup_1} [debug] Received data from Time Control Plugin: EnablePlugin;1

[2024-11-15 12:36:27.803] {afsimbridge_ServiceGroup_1} [debug] Time Control Plugin responded to EnablePlugin: true
Created DIS entity.
    T = 0
    Entity: 1:1:1
    Type: 1:1:225:1:1:1:1
    Force: 0 (local)
    WSF Name: radar_1
    WSF Type: ACQ_RADAR
    WSF Side: blue
Created DIS entity.
    T = 0
    Entity: 1:1:2
    Type: 1:1:225:1:1:1:1
    Force: 0 (local)
    WSF Name: radar_2
    WSF Type: ACQ_RADAR
    WSF Side: blue
Created DIS entity.
    T = 0
