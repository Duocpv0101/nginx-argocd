# Use the official Python image as the base image
FROM python:3.14.0a3-slim

# Set the working directory to /app
WORKDIR /app

# Copy the requirements file into the container
COPY requirements.txt .

# Install the Python dependencies
RUN pip install --no-cache-dir -r requirements.txt

# Copy the Python script into the container
COPY app.py .

# Expose the port that the Flask app will run on
EXPOSE 8081

# Set the command to run the Flask app
CMD ["python", "app.py"]