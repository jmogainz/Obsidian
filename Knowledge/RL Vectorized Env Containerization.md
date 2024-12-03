- docker or singularity from python
- in init, generate unique port for communication
	- create container with the port mapping, spinning up a simulation controller service that facilitates communication between the Gym environment and the BatchController/Environment 
- step function, instead of calling the functions it needs to directly, sends message commands to simulation controller
- simulation controller receives message commands on fixed port (8080) and based on the message command (enum), will call certain functions of the batch controller /environment
- gym env step function will call the step of the environment, this will make actions, get observation, and advance logical time
- gym env reset will call nextScenario() and then startScenarioAndWait()
- use the env config to determine what number worker and what scenarios to grab from scenario pool
```python
```python
import docker
import random

client = docker.from_env()

def allocate_host_port():
    # Just an example; you'd probably want a more robust way to allocate ports
    return random.randint(30000, 40000)

def start_simulation():
    host_port = allocate_host_port()
    container_port = '8080/udp'  # Replace with the actual port your simulation listens on
    
    container = client.containers.run(
        "simulation_image", 
        detach=True, 
        ports={container_port: host_port}
    )
    
    return container, host_port
```
```