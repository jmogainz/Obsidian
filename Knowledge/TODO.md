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

POOF
- [X] Auth service security incorporation from chatgpt backend development convo
- [X] full health check support from chatgpt integrated, make sure endpoint is in backend
- [x] inline the auth service more for our actual use case on the front end now and what we are going to need
- [ ] Put the proper tokens, api keys, etc inside my .env folder for my auth service
- device tokens implement
- Ask about needing redis, specifically flyes upstash redis, in auth service
- Integrate nginx for starting on services on a single node. Once I distribute across machines (multiple fly apps for scalability), I dont need it to route to do path based routing. Fly will also do load balancing for me across the different apps. At the start, that load balancing will of course only between the nginx server and the front end app.
- Setup docker compose with database and auth service
- Create docker file for the auth service and make sure the auth service builds successfully inside and all unit tests pass
- Time to see if our database migrations work at all.....
- Write integration tests between auth service and database, run them and test them.
- Perform load testing locally? Maybe not. Depoy to fly and test on there in the free tier if we can?


RESUME
- [ ] Experience completion, skills completion, and then finally, add poof section to projects

Temp promt:

I work for a startup company where I am a founder and the CTO. This is a tech company, and the lead developer. How would you suggest that I can include this work that I do for this in some way on my resume that is ethical...as this is an active/recognized corporation in any way yet. Perhaps this may not matter, but I am wondering what is the recommended way to include this in some way on my resume as I am still working full time, and our startup is in primarily development phase for now, and we keep the idea the company purpose private to the founders until we hit launch. But in the meantime I am trying to move full time jobs and think the experience with the company would benefit my resume greatly.
