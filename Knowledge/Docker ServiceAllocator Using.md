1.  Build images
2.  Customize serviceallocator yaml
3.  IF RUN LOCAL: modify afsimbridge local-template json
4.  IF RUN REMOTE METHOD 3: run ServiceAllocator with cmd line args for the amount of service groups you need to spin up (vectorized environment size)
5. If RUN REMOTE: docker logs
6. If RUN LOCAL: logs are in bin
7.  IF RUN REMOTE: stop containers, prune containers
8.  IF RUN REMOTE: remove docker networks

##### Docker Helper Commands
- docker ps -a --> show containers
- docker images  --> show images
- docker stop $(docker ps -aq) --> stop containers
- docker container prune --> delete containers
- docker builder prune --> delete build cash
- docker system prune --> delete everything
- docker image prune --> delete images
- docker system df --> check categorized docker storage usage
- docker run -it --entrypoint /bin/bash <img:latest> --> test docker image
- docker network ls --> see all networks
- docker network rm *name* --> delete network
- docker inspect *container* --> check container info
- docker network inspect *network* --> check network info
- docker network prune --> removes all unused networks