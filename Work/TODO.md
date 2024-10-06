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

WHY ARE THERE TWO LOGS COMING FROM PYTHON
Overloaded function support for lua plugin c++
Jeremy uh oh. We need that path to go into the rpath of the resulting libs and execs

- Look into csum_ipv6_magic, and try to find the same performance problem that Eric and Linus were talking about here: https://lkml.org/lkml/2024/1/6/180
