# Docker Basics

## Review the python script
1. This is a script that has a function that takes text and reverses the order
1. Take note of the version of python that you are using (in this case, it is 3.9)
1. Create a requirements file by running the following command from in this folder:
    `pip freeze > requirements.txt`
    1. This command creates a file called requirements.txt that will list all libraries in the current environment

## Setting up a Basic Docker File
1. Create a file called simply `Dockerfile`
1. Into the file, input the following
    ```
    # Use an official Python runtime as a parent image
    FROM python:3.9-slim

    # Set the working directory in the container
    WORKDIR /usr/src/app

    # Copy the requirements file into the container at /usr/src/app
    COPY requirements.txt .

    # Install any needed packages specified in requirements.txt
    RUN pip install --no-cache-dir -r requirements.txt

    # Copy the rest of the current directory contents into the container at /usr/src/app
    COPY reverse_text.py .

    # Run the script when the container launches
    CMD ["python", "./reverse_text.py"]
    ```

## Review the Dockerfile
1. ```FROM python:3.9-slim```
    1. This line tells docker which image is the base environment that everything else will be built on.
    1. In this case, we are using an install of python 3.9.
    1. The slim means that it is installed with a minimum of additional libraries
1. ```WORKDIR /usr/src/app```
    1. This sets the place within the container's filesystem that we will be operating.
    1. From this point forward, files that are copied to `.` can be seen as though they are copied to `/usr/src/app`
1. ```COPY requirements.txt .```
    1. This line copies the file you created called requirements.txt into the docker at the appropriate place to be used by the app
1. ```RUN pip install --no-cache-dir -r requirements.txt```
    1. This line uses the requirements file that you just created and copied into the container to install all the python packages that are needed to run the program
1. ```COPY reverse_text.py .```
    1. This copies the main python script reverse.py into the container's working directory
1. ```CMD ["python", "./reverse_text.py"]```
    1. The CMD command runs commands from the bash command line, taking commands as a list.  THis list is concatonated together before running, iwth a space inserted between commands.
    1. In this case, `CMD ["python", "./reverse_text.py"]` is the same as running `python ./reverse_text.py` from the BASH prompt.

## Build, Run, Stop, and Delete the Image
1. Make sure that your Docker Daemon is up and running
    1. The exact method to do this will depend on your operating system
1. Open a powershell or BASH prompt and navigate to the same directory with the Dockerfile
1. Build the image using the following command `docker build -t reverse-text .`
    1. The `-t` argument indicates that the value after it is a tag.  These are not necessarily needed, but is recommended for traceability reasons.
    1. The `.` at the end tells docker where to build the image.
1. Use the command `docker images` to list what images are on your system. You should see one with the tag `reverse-text` in the `REPOSITORY` column
2. Now, run the container using the command `docker run reverse-text`
    1. This will execute the program `reverse_text.py` from within the container, meaning that it it will work even if you do not have python 3.9 on your computer.
3. Now run `docker ps -a`.  This will list all images that are on your system.
    1. The `-a` argument will list all containers, irregardless of status.  Without this argument, it will only display currently running containers
4. Now lets clean up our system
    1. `docker container prune` will remove all containers that aren't currently running.
    2. `docker image prune -a` will delete all images that aren't being used by running containers