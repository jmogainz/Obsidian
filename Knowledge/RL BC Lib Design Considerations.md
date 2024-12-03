[[RL Research]]

- main: constructor, initialization, shutdown
	- executable use of BC
	- constructor takes scenarioManager as arg
	- initializer takes in StructuredData config object
	- Build readers
	- shutdown batch controller
	- what about argc and argv?

- replay scenarios
	- runScenarioManagement()
		- loops the startScenario() managing the test cases
		- uninterrupted sequential
	- startScenario()
		- manages the run of a scenario and the time advance
			- time advance is managed by our rl environment being a registered service
	- !testDriver_->moveToNextTestCase()
		- move our iterator pointer to next case, increment