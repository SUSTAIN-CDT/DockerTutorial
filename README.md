# DockerTutorial
Docker is a powerful platform for containerization that allows developers to package applications and their dependencies into portable, lightweight containers. These containers ensure consistent environments across different systems, eliminating issues like "it works on my machine" by encapsulating the application, libraries, and configurations.

Docker simplifies deployment, enhances reproducibility, and isolates environments, making it ideal for development, testing, and production. 

By combining Docker with Conda, a package and environment manager, you can create a reproducible Python environment with specific libraries like PyTorch, which can be shared with others, ensuring they can run the same setup without compatibility issues.

Below, I'll provide a step-by-step guide to setting up Docker and creating a basic Conda environment with PyTorch inside a Docker container.


# Docker Installation


# In your console
# Use an official Ubuntu base image for compatibility and familiarity
FROM ubuntu:20.04

# Set environment variables to avoid interactive prompts during installation
ENV DEBIAN_FRONTEND=noninteractive

# Update package lists and install essential tools
RUN apt-get update && apt-get install -y \
    wget \
    curl \
    git \
    && rm -rf /var/lib/apt/lists/*

# Download and install Miniconda
RUN wget https://repo.anaconda.com/miniconda/Miniconda3-latest-Linux-x86_64.sh -O /tmp/miniconda.sh && \
    bash /tmp/miniconda.sh -b -p /opt/conda && \
    rm /tmp/miniconda.sh

# Add Conda to PATH for easy access
ENV PATH=/opt/conda/bin:$PATH

# Initialize Conda for bash
RUN /opt/conda/bin/conda init bash

# Create a Conda environment named 'pytorch_env' with Python 3.8
RUN conda create -n pytorch_env python=3.8 -y

# Activate the Conda environment and install PyTorch
RUN /bin/bash -c "source /opt/conda/bin/activate pytorch_env && \
    conda install pytorch torchvision torchaudio cudatoolkit=11.3 -c pytorch -y"

# Set the default command to open a bash shell in the Conda environment
CMD ["/bin/bash", "-c", "source /opt/conda/bin/activate pytorch_env && bash"]



Sharing the Conda Environment in the Docker Container
The Conda environment (pytorch_env) is created within the Docker container during the build process, as defined in the Dockerfile. When you share the Docker image (e.g., by pushing it to a registry like Docker Hub or exporting it as a .tar file), the Conda environment, including Python 3.8, PyTorch, and its dependencies, is encapsulated within the image.
