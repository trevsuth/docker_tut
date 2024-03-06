# Docker Basics pt 2

## Background
1. This picks up where Lesson 1 left off and begins using the same Dockerfile and python script.

## Working in a container
1. It is possible to work from within a container 
    1. Open a command prompt to the directory that contains this file
    1. Now build a container and run it, but when running the container use the -it (interactive) argument and specify the BASH shell 
    ```
    docker build -t reverse-text .
    docker run -it reverse-text bash
    ```
    2. This will give you command line access to the container running python script.
1. Notice that your command prompt has changed to something like `root@ebe7d23e967d:/usr/src/app#`.  This means that you are working inside the container, in this case container id ebe7d23e967d.  Take note of the ID of the container that you are in.
1. From here, use the `pwd` (print working directory) command to get your bearing inside the container.
    1. You will see that it returns /`usr/src/app`.  This is the directory that you specified as the `WORKDIR` in the `Dockerfile`
1. Now type `ls` to list all files in the directory.  You will see that `reverse_text.py` is here, but there is no text editor here to edit it.
    1. We could add it from here, but better practice it to modify the `Dockerfile` to include this 
1. Type `exit` to leave the container and return to your local operating system

## Adding more programs to your Docker Image
1. Make a copy of your `Dockerfile` and call it `Dockerfile_edit`
1. After the line where you specify the `WORKDIR`, add the following to a new line:
    ```
    # Install nano
    RUN apt-get update && apt-get install -y nano && rm -rf /var/lib/apt/lists/*
    ```
1. This line tells the linux operating system that is in your docker container to install the `nano` text editor and then to remove some of the additional files that are created while doing that update

Your new file should look like this:
```
# Use an official Python runtime as a parent image
FROM python:3.9-slim

# Set the working directory in the container
WORKDIR /usr/src/app

# Install nano
RUN apt-get update && apt-get install -y nano && rm -rf /var/lib/apt/lists/*

# Copy the current directory contents into the container at /usr/src/app
COPY reverse_text.py .

# Run the script when the container launches
CMD ["python", "./reverse_text.py"]
```

## Making a new image
1. Since there are now 2 dockerfiles in our directory, we need to specify which we want to use when building our image.  
1. Run the following command: `docker build -t new-image -f Dockerfile_edit .`
    1. The `-f` argument tells docker to use the value after it as the Dockerfile, in this case our edited file
1. Now run the container in interactive mode `docker run -it new-image bash`
1. Once you are in the container, type `nano reverse_text.py` to open the python file in the editor.
1. Now, change text to be reversed in the script to say "Tis but a flesh wound", then press Ctrl+O to save the change and Ctrl+X to leave the editor.
1. Run the program by typing `python reverse_text.py` and see the output has changed.
1. Leave the container by typing `exit`
1. Now run the container normally to `docker run new-image` and note the output: It did not change.
1. To understand what happened, run the command `docker ps -a` to see your containers
1. Notice that there are multiple containers that are using the new-image tag; as you run the same image multiple times, it will create a new container.
    1. While there are ways to reactivate an existing container, it's usually not the best practice.
    2. Instead, if you will be changing values passed to your program you might be better off doing this using environment variables

## Creating environment variables
1. Environment variables are usually passed to your programs using a `.env` file or from your operating system.  Since a docker container is basically a mini-operating system, we can create these from our dockerfile
1. First we will need to create a copy of our reverse_text.py file and change it to take text from an environment variable
    1. Copy `reverse_text.py`.  Call this copy `env_text.py`
    1. Now, we will need to import the os library by adding `import os` as the first line of the script
    1. Lastly, alter the line `text = "Hello, world!"` to say instead `text = os.getenv('MY_TEXT', 'Hello, world!')`
        1. This will tell the program to look for an environment variable called MY_TEXT and to use that value.  If MY_TEXT does not exist, it will default to "Hello World"
    1. Your new program should look like this
    ```
    import os

    # Define a function to reverse the order of characters in a text
    def reverse_text(text):
        # Reverse the text and return
        return text[::-1]

    # Example text to reverse
    text = os.getenv('MY_TEXT', 'Hello, world!')
    # Call the function with the example text
    reversed_text = reverse_text(text)
    # Print the reversed text
    print(reversed_text)
    ```
1. Next we will need to change our dockerfile to run env_text.py and also to supply the environment variables
    1. Copy your original dockerfile (`Dockerfile`).  Call this copy `Dockerfile_env`
    1. Add the following line to your Dockerfile_env file after the line where you assign the WORKDIR: 
    ```
    # Set the environment variables
    ENV MY_TEXT "Tis but a flesh wound"    
    ```
    1. Next, change the COPY command so that it imports `env_text.py` 
    1. Lastly, change the final line so that it runs `env_text.py` and not `reverse_text.py` 
    1. The final file should look like this:
    ```
    # Use an official Python runtime as a parent image
    FROM python:3.9-slim

    # Set the working directory in the container
    WORKDIR /usr/src/app

    # Set the environment variables
    ENV MY_TEXT "Tis but a flesh wound" 

    # Copy the current directory contents into the container at /usr/src/app
    COPY env_text.py .

    # Run the script when the container launches
    CMD ["python", "./env_text.py"]
    ```
1. Build and run this new dockerfile and see what it outputs
    ```
    docker build -t env_image -f Dockerfile_env .
    docker run env_image
    ```
    1. The output should be based on the text that you gave in Dockerfile_env
1. NOTE that if you change the value of MY_TEXT in the dockerfile, you will need to re-build the image for this change to re recognized.
    1. If you will be frequently changing this value, there are better ways to do this

## Mounting containers to your local filesystem
1. If you have values that will be changing frequently, for example data sets, it might be better to write your program to reference a file, then bring it "into" your local file system by mounting it to your local filesystem.  
    1. This sounds complicated, but it really isn't
1. First, we will want to change `reverse_text.py` so that the value of text is loaded from an external file
    1. Copy reverse_text.py in to a file called `reverse_from_file.py`
    1. Change the line `text = "Hello, world!"` to open the file `file.txt`.
        ```
        with open('file.txt', 'r') as file:
            text = file.read().strip()
        ```
        1. The updated file should look like this:
            ```
            # Define a function to reverse the order of characters in a text
            def reverse_text(text):
                # Reverse the text and return
                return text[::-1]

            # Example text to reverse
            with open('file.txt', 'r') as file:
                text = file.read().strip()

            # Call the function with the example text
            reversed_text = reverse_text(text)

            # Print the reversed text
            print(reversed_text)
            ```
        1. Now create a new file called file.txt
            1. the contents of the file should read: "We apologize again for the fault in the subtitles. Those responsible for sacking the people who have just been sacked have been sacked."
        1. Lastly, we need to change our dockerfile to also incorporate our new text file
            1. Copy your original dockerfile (`Dockerfile`).  Call this copy `Dockerfile_external`
            1. Change the COPY portion of the file to copy both reverse_from_file.py and file.txt.
                ```
                COPY reverse_from_file.py .
                COPY file.txt .
                ```
            1. Lastly, change the final line of the file to run `reverse_from_file.py` instead of `reverse_text.py`.  The final file should look like this:
                ```
                # Use an official Python runtime as a parent image
                FROM python:3.9-slim

                # Set the working directory in the container
                WORKDIR /usr/src/app

                # Copy the current directory contents into the container at /usr/src/app
                COPY reverse_from_file.py .
                COPY file.txt .

                # Run the script when the container launches
                CMD ["python", "./reverse_from_file.py"]
                ```
        1. Build and run your container
            ```
            docker build -t ext_image -f Dockerfile_external .
            docker run ext_image
            ```
            It should display the text from the file reversed
1. Now we want to run this container in a way that the code in the container will run on files in our local file system.  To test this let's make another text file called `local_text.txt`.
    1. Inside this file, put the following text: "I don’t want to talk to you no more, you empty-headed animal food trough wiper. I fart in your general direction. Your mother was a hamster and your father smelt of elderberries."
1. Now we are going to run the docker container again, but this time we are going to use the -v argument to mount files from your local directory to the container
    1. This uses the general pattern of `docker run -v <local_file>:<container_file> <image_name>`
    1. So, in my case I will run: `docker run -v ~/Code/docker_tut/Lesson2/local_text.txt:/usr/src/app/file.txt ext_image`, though the first part, the local file portion, will depend on how you computer is set up.
        1. The container_file always needs to be a full filepath, from the root `/` to the end of the file name
1. When we run this command, we will see that the text returned is based on that from `local_text.txt`, which was not in the container.
    1. Furthermore, we can change the contents of this file at any time and re-run the command with the `-v` flag and it will use the new values of that file without re-building the image.
    1. This is especially handy in creating data pipelines.

## Cleanup
1. Before we are done, we need to dispose of all the images and containers that we have made.  Run the following
    ```
    docker container prune
    docker image prune -a
    ```
1. Lastly, confirm that everything is deleted by running the following commands:
    ```
    docker ps -a
    docker images
    ```