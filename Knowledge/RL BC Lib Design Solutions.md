[[RL BC Lib Design Considerations]]

- main: constructor, initialization, shutdown
	- library use of BC
	- overloaded initializer
		- takes in:
			- app name
			- app instance
			- app environ
		- does:
			- creates scenario manager and sets member instance
			- creates StructuredData config
			- everything initialize currently does

- replay scenarios
	- custom scenario management that can manage test cases differently
		- Wrap pre-launch tasks
			- prepareForLaunch()
				- Create and set directory path for test log files
				- Verify test driver has an eligible test ready to run
				- oncePerTestSetup()
		- Wrap start and wait of scenario
			- startScenario()
		- Wrap next test case
			- nextScenario()