### Build and Run
* `docker build -t <tag-name> .` Build an image
* `docker run <tag-name>` Run an image

### Administer Images on System
* `docker images` List images on system
* `docker ps -a` List all containers on system

### Stopping and removing containers
* `docker container prune` Remove all containers that are not running
* `docker image prune -a` Remove all images not used by running containers