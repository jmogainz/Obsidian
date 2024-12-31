Okay can you now help me setup all of these extra endpoints you have just suggested and add these to my auth service? Lets start with signup (register) for property_manager role and worker role. 

Here is the information I am aware of that needs to come in from the frontend for registration as of now. Make sure you allow for many other fields to come in as well in the future as we get a better idea:

Worker:
First name, last name, email, mobile phone number, 

street address, apt/suite, city, state, zip code,  

vehicle year, vehicle make, vehicle model, 

18 years of age certification, etc. 

We will keep adding more from here. And please actually logically divide these into /worker/register/<type> endpoints as the front end will send this info in groups as the user goes logically separate group (page) by group, so that I can do incremental saving during the registration process.

Property Manager:
first name, last name, email, number (optional)

Business name, business address, city, state, zip code

Property management agreement or authorization letter, persona verification information (persona would give this to us right?), proof of business registration such as business license.

Property buildings addresses, every single unit number

Trash compactors coordinates (for each one)








