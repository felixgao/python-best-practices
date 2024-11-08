# Docker Best Practice

In the realm of machine learning, reproducibility is paramount. Ensuring that experiments can be replicated reliably across different environments is crucial for scientific rigor and collaboration. Docker, a powerful containerization tool, has emerged as a vital component in modern ML workflows. By encapsulating applications and their dependencies into standardized containers, Docker provides a consistent and portable environment for ML development, deployment, and collaboration.

## General Best Practices

### Use Officil Base Images
Start with official Python images to ensure compatibility and security.  It is important to use the latest Python version to ensure the system is up to date. 

```dockerfile
FROM python:3.12-slim
```


### Use a .dockerignore File
Exclude unnecessary files from the build context.

```text
__pycache__
*.pyc
*.pyo
*.pyd
.Python
env
pip-log.txt
pip-delete-this-directory.txt
.tox
.coverage
.coverage.*
.cache
nosetests.xml
coverage.xml
*.cover
*.log
.git
.mypy_cache
.pytest_cache
.hypothesis
```

### Minimize the Number of Layers
Combine related commands into a single RUN instruction to reduce the number of layers and image size.
Intead of running the `RUN` command on every input, you can combine them to create a single layer. 

```dockerfile
FROM ubuntu:20.04
RUN apt-get update && apt-get install -y \
    python3 \
    python3-pip \
    && rm -rf /var/lib/apt/lists/*
```

### Run as Non-Root User
Container escaping is possible due to bugs in the code/host.  Create and use a non-root user for better security. 

```dockerfile
RUN adduser --disabled-password --gecos '' appuser
USER appuser
WORKDIR /home/appuser/app
```

### Use Enviornment Variables and Arguments
Environment variables and arguments in Dockerfiles are powerful tools for creating flexible and customizable container images. They allow you to dynamically configure your application's behavior without rebuilding the image. This is particularly useful in machine learning workflows where you may need to experiment with different hyperparameters, datasets, or models.

```dockerfile
# Build-time argument for the base image
ARG PYTHON_VERSION=3.11-slim-buster

# Base image
FROM python:${PYTHON_VERSION}

# Build-time argument for the dataset URL
ARG DATASET_URL=https://example.com/dataset.zip

ARG MODEL_ENV=test
# Environment variables
ENV PYTHONUNBUFFERED 1
ENV MODEL_DIR /model
ENV DATASET_DIR /data
ENV MODEL_ENV=$MODEL_ENV

# Set the working directory
WORKDIR /app

# Download and extract the dataset (using a script)
RUN bash -c "ds_loader -q $DATASET_URL && unzip dataset.zip -d $DATASET_DIR"

# Install dependencies
COPY requirements.txt requirements.txt
RUN pip install -r requirements.txt

# Copy the application code
COPY . .

# Define the command to run the training script
CMD ["python", "train.py", "--data_dir", "$DATASET_DIR", "--model_dir", "$MODEL_DIR"]
```

In the above example. The user could control the version of the `PYTHON_VERSION` and the `MODEL_ENV` parameter to alter the code behavior in `train.py`

an example use would be

`docker build --build-arg PYTHON_VERSION=latest --build-arg MODEL_ENV=production --no-cache .`
This example changes the base docker to the latest python version and set the environment variable to production.

### Optimize Caching
Order Dockerfile instructions from least to most frequently changing to optimize the caching.  This will result in a faster docker build when changes needs to happen.

```dockerfile
FROM python:3.12-slim
WORKDIR /app
COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt
COPY . .
CMD ["python", "app.py"]
```

In the above example we assume `requirements.txt` is relatively stable only the application code is changing often. Then all of the dependencies installed by the pip command is already cached in the layer that will not need to be re-installed again

###  Use Healthchecks

A Docker healthcheck is a useful tool to monitor the health of a container. For long-running training jobs, a custom healthcheck can provide valuable insights into the job's progress and potential issues. This is not limited to the long running tasks, it can be used to attempt to reach an application endpoint or check a process’s status. If the health check command returns a failure due to an application crash, the container is marked as unhealthy. You can then restart the container or stop it and shift traffic to other instances.

```dockerfile
FROM tensorflow/tensorflow:latest

WORKDIR /app

# dependencies installed for the job
# COPY requirements.txt requirements.txt
# RUN pip install -r requirements.txt

COPY . .

# ... other build instructions ...

HEALTHCHECK --interval=30s --timeout=10s CMD if [ -f /app/train.log ] && [ $(stat -c %s /app/train.log) -gt 0 ]; then exit 0; else exit 1; fi
```
The above example checks the health every 30 seconds, times out the check after 10 seconds. The command to use for checking the health is checks if the train.log file exists and the file size is greater than 0. If both conditions are met, the healthcheck passes.


## ML Docker Container Best Practice

As an opinonated guide, I highly recommend to use [cog] to create your ML docker container

### Use GPU-enabled Base Images.
For ML projects requiring GPU, use CUDA-enabled base images.  Whenever possible we should try to leverage the official base images because security fixes. 

```dockerfile
FROM nvidia/cuda:11.3.1-runtime-ubuntu20.04
``` 

### Install ML-specific libraries
Install commonly used ML libraries efficiently. To avoid cuda hell with other well known libraries, it is recommended to install the commonly used libraries for ML into the container.

```dockerfile
RUN pip install --no-cache-dir numpy pandas scikit-learn torch torchvision
```

Note: the above didn't specify the version number of each dependnecies.  It is recommended to pin to a version instead of relying on pip to figure out the latest.

### Manage Large Model Files or datasets
Use volume mounts for large model files instead of including them in the image.

```dockerfile
VOLUME /app/models
VOLUME /app/datasets
```

This assumes the models or dataset could use `init-container` to be downloaded.  


### Optimize for Inference
For inference containers, focus on runtime dependencies only.

```dockerfile
FROM python:3.12-slim
COPY --from=builder /app/model /app/model
COPY requirements-inference.txt .
RUN pip install --no-cache-dir -r requirements-inference.txt
COPY inference.py .
CMD ["python", "inference.py"]
```

Only install what is necessary for inference.

### Version Control for Models and Data
Use version control systems for models and data, and reference specific versions in your Dockerfile.

```dockerfile
ARG MODEL_VERSION=v1.2.3
RUN wget https://model-repo.com/model-${MODEL_VERSION}.pkl -O /app/model.pkl
```

### Use Appropriate Concurrency
Set the number of workers based on available resources.

```dockerfile
CMD gunicorn --workers 4 --threads 4 --bind 0.0.0.0:8000 app:app
```
Since python is under 3.13 still have GIL in place. The command above assume you have a quad core(4) CPU and each core have hyperthreading enabled to fill the instruction pipeline of the CPU.

### Optimize memory 

```dockerfile
# Use an official Python runtime as a parent image
FROM python:3.12-slim

# Set environment variables for memory optimization
ENV PYTHONMALLOC=malloc
ENV PYTHONMALLOCSTATS=1
ENV MALLOC_TRIM_THRESHOLD_=100000
ENV PYTHONHASHSEED=0
ENV PYTHONASYNCIODEBUG=1
ENV PYTHONTRACEMALLOC=1
ENV PYTHONDEVMODE=1
ENV PYTHONMEMORY=4294967296  # Set memory limit to 4GB
ENV PYTHONGC=1
ENV OMP_NUM_THREADS=4

...

# Run app.py when the container launches
CMD ["python", "app.py"]
```
The above is just an example, your usage may vary.

Here's a breakdown of the environment variables used:
- PYTHONMALLOC=malloc: Uses the standard C malloc instead of Python's custom allocator.
- PYTHONMALLOCSTATS=1: Prints memory allocation statistics.
- MALLOC_TRIM_THRESHOLD_=100000: Sets a lower threshold for releasing memory back to the system.
- PYTHONHASHSEED=0: Makes memory usage more predictable across runs.
- PYTHONASYNCIODEBUG=1: Enables debugging mode for asyncio to help identify memory leaks in asynchronous code.
- PYTHONTRACEMALLOC=1: Enables tracemalloc to track memory allocations.
- PYTHONDEVMODE=1: Enables additional checks that can help catch memory-related issues earlier.
- PYTHONMEMORY=4294967296: Sets a memory limit of 4GB for Python processes.
- PYTHONGC=1: Enables garbage collection.
- OMP_NUM_THREADS=4: Limits OpenMP to 4 threads, which can help control memory usage in scientific computing libraries.


[cog]: https://cog.run/