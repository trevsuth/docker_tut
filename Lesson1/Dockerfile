# Use an official Python runtime as a parent image
FROM python:3.9-slim

# Set the working directory in the container
WORKDIR /usr/src/app

# Copy the current directory contents into the container at /usr/src/app
COPY reverse_text.py .

# Run the script when the container launches
CMD ["python", "./reverse_text.py"]
