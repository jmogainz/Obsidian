- [x] Move extractioninstructionparser to its own lib
- [ ] Extraction instruction grammar documentation
- [ ] Design data recorder
- [ ] Fix clairvoyanttrackwriter json field in commandlinetestdriver? The name is wrong on it
- [ ] Catch errors in python calls through c++ wrappers to handle and shutdown the pipeline
- [ ] Apply clairvoyant sensor data observer filter to sensor limits
- [ ] Make sure that inMemory is checked on the register mapping to determine if we need to load into temp register or not
- [ ] Test with multiple files to check assembly output and see if it works and looks right
- [ ] Implement tests for code generation and make sure output matches that which was interpreted
- [ ] Pull safeSet into its library


**Pressing**
- [ ] Should we duplicate the flatbuffers dependency
- [ ] Potentially hyperlink uml diagrams
- [x] Look over data recorder library code to ensure everything looks right
- [x] Consider what daniel says about recursive variant potentially being a better option
- [ ] Get rid of 'schema' and other flatbuffer related language from datarecorder library

Take memory from windows partition and give it to my vm
Have datarecorder have a default output directory
WHY ARE THERE TWO LOGS COMING FROM PYTHON
Overloaded function support for lua plugin c++
Jeremy uh oh. We need that path to go into the rpath of the resulting libs and execs

- Look into csum_ipv6_magic, and try to find the same performance problem that Eric and Linus were talking about here: https://lkml.org/lkml/2024/1/6/180
1. Check with daniel with validation of paths for env config in pydantic model
2. Make evals from the yaml config more robust

Add tensorboard log to models
Add annotated types to models


HERE:
we need code improvements on auth (in apple notes)
	- endpoints that strictly check if the email or phone number exist (registered with a user profile already, two separate end points).....this should use code that is reusable by the login enpoints as they need to perform similar logic to check if the username exist. 
	- additionally we need full web abilities as well for requests that come from the web app version of the app
	- 
containerize dev environment

