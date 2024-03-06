### Basic Build and Run
* `docker build -t <tag-name> .` Build an image
* `docker run <tag-name>` Run an image

### More Build and Run
* `docker build -t <tag-name> -f <filename> .` Build an image using a specific dockerfile
* `docker run -it <tag-name> <command>` Run an image in interactive mode using the given command
* `docker run -v <local_file>:<container_file> <image_name>` Run an image using a local file

### Administer Images on System
* `docker images` List images on system
* `docker ps -a` List all containers on system

### Stopping and removing containers
* `docker container prune` Remove all containers that are not running
* `docker image prune -a` Remove all images not used by running containers