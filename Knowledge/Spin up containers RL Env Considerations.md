[[RL Research]]

C++ Library
Service Allocator
----------------------------
###### ServiceAllocator.cpp
    allocate()
	allocate_unique_ports()
###### ConfigGenerator.cpp
	generate()
	
- Config driven for different modes
	- Docker (system->docker & docker->docker)
		- Spins up containers
			- Can we create containers from within a container? No....
			- **This (prob 3) would require up-front container creation no?**
	- Non-Docker
		- Starts executables

Mode 1
Problem 2
- Dynamically create docker/singularity containers within each gym env
	- They need to be port mapped to generate unique ports based on allocate_unique_ports
	- They need to start AfsimBridge and all other necessary services
- **New config file for BC and Env with unique ports each time**
	- Port count
		- 42 for non-docker mode
		- 15 for system->docker mode
		- 15 for docker->docker mode
	- **Where will these new generations go?**
		- Perhaps in the highest level possible
			- Perhaps in the TBOX_CONFIG_PATH
			- Perhaps in the current folder
			- Perhaps in the project root
	- New appInstance each time
		- env1, env2, env3, ...
	- appEnviron
- Start up EnvWrapper with appInstance env<worker_id>
- Start up BCWrapper with appInstance env<worker_id>

Mode 2
Problem 1
- TWO OPTIONS
	1. **Dynamically allocate 42 unique ports, and generate unique config files for each service needed**
	2. Run off of already created config files, create them custom
	   
	   
   TODO
   - [x] Add logging in
   - [x] Put up merge request
   - [x] Resolve daniels threads on merge request
   - [ ] Done with local
   - [ ] Do remote
	   - [ ] Afsim will need to be set in template (not a responsibility of service allocator)
   - [ ] Setup networks for docker running
   - [x] Setup toolbox service image
   - [ ] Get scenarios into rlsim repo
   - [ ] Get docker image to contain all necessary scenarios
   - [ ] Copy over files from matt so I can get them in docker image
   - [ ] Check batchcontroller where it is looking for scenarios
   - [ ] Set up port mapping so that networks can talk to host machine
   - [ ] Figure out why batchController is being gay
   - [ ] OH CRAP....this not good....ports are jacked up bad...not gonna work
   - [ ] Make an image for each toolbox service for more lightweight
   - [ ] Copy in generated template configs
   - [ ] Should of written service allocator so that each mode was parsed, stored, and run through completely different strategies
   - [ ] Make sure to update all templates
   - [ ] Map up jsons
   - [ ] Move things pertaining to remote config generator strategy to private functions within
   - [ ] Set up port mapping into docker command
   - [ ] Set up ip address into docker command
   - [ ] Mount in created executables
   - [ ] Update instance launch of docker run command
   - [ ] Create custom handling in remote docker run for log_level logging the starting of services (detached if no logging necessary)
   - [ ] Refactor lists to just be maps not lists of strings (config file name to )?
   - [ ] Possibly refactor to have overloaded updatePorts function
   - [ ] update protected to private in igenerator for better readability
   - [ ] refactor

# DO NEXT
1. ONLY MAP PORTS THAT WRITE TO CONTAINER
	1. ip address set as container
2. WRITE names of each json as ip 
3. ADD CODE THAT REPLACES NAMES
	1. virtual update local
4. MOUNT CREATED JSON CONFIGS IN
5. CHANGE INSTANCE ARG TO ServiceGroup_<worker_id>

- Options for solving ip issue
	- Let names go through dns for ip? Probably have to rewrite how connections factory is done
	- Pre-map and then Increment every IP address for each worker
	- Run everything in a single container
   