# Docker Tutorial: Setting up Conda and PyTorch in Containers

Docker is a powerful platform for containerization that allows developers to package applications and their dependencies into portable, lightweight containers. These containers ensure consistent environments across different systems, eliminating issues like "it works on my machine" by encapsulating the application, libraries, and configurations.

Docker simplifies deployment, enhances reproducibility, and isolates environments, making it ideal for development, testing, and production. By combining Docker with Conda, a package and environment manager, you can create a reproducible Python environment with specific libraries like PyTorch, which can be shared with others, ensuring they can run the same setup without compatibility issues.

## Docker Installation

### Linux Installation

#### Ubuntu/Debian
```bash
# Update package index
sudo apt-get update

# Install required packages
sudo apt-get install apt-transport-https ca-certificates curl gnupg lsb-release

# Add Docker's official GPG key
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /usr/share/keyrings/docker-archive-keyring.gpg

# Set up the stable repository
echo "deb [arch=amd64 signed-by=/usr/share/keyrings/docker-archive-keyring.gpg] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null

# Install Docker Engine
sudo apt-get update
sudo apt-get install docker-ce docker-ce-cli containerd.io

# Add your user to the docker group to run Docker without sudo
sudo usermod -aG docker $USER

# Start and enable Docker service
sudo systemctl start docker
sudo systemctl enable docker
```

#### CentOS/RHEL/Fedora
```bash
# Install Docker using dnf (Fedora) or yum (CentOS/RHEL)
sudo dnf install docker  # For Fedora
# or
sudo yum install docker  # For CentOS/RHEL

# Start and enable Docker service
sudo systemctl start docker
sudo systemctl enable docker

# Add your user to the docker group
sudo usermod -aG docker $USER
```

### Windows Installation

1. **Download Docker Desktop for Windows** from the official Docker website: https://www.docker.com/products/docker-desktop
2. **Run the installer** and follow the installation wizard
3. **Enable WSL 2** (Windows Subsystem for Linux) if prompted - this is recommended for better performance
4. **Restart your computer** when installation is complete
5. **Launch Docker Desktop** from the Start menu
6. **Verify installation** by opening PowerShell or Command Prompt and running:
   ```cmd
   docker --version
   ```

### macOS Installation

1. **Download Docker Desktop for Mac** from: https://www.docker.com/products/docker-desktop
2. **Open the .dmg file** and drag Docker to your Applications folder
3. **Launch Docker Desktop** from Applications
4. **Verify installation** by opening Terminal and running:
   ```bash
   docker --version
   ```

## Creating the Dockerfile

Create a new file named `Dockerfile` (no extension) in your project directory and add the following content:

```dockerfile
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
RUN wget https://repo.anaconda.com/miniconda/Miniconda3-latest-Linux-x86_64.sh -O /tmp/miniconda.sh \
    && bash /tmp/miniconda.sh -b -p /opt/conda \
    && rm /tmp/miniconda.sh

# Add Conda to PATH for easy access
ENV PATH=/opt/conda/bin:$PATH

# Initialize Conda for bash
RUN /opt/conda/bin/conda init bash

# Create a Conda environment named 'pytorch_env' with Python 3.8
RUN conda create -n pytorch_env python=3.8 -y

# Activate the Conda environment and install PyTorch
RUN /bin/bash -c "source /opt/conda/bin/activate pytorch_env && \
    conda install pytorch torchvision torchaudio cudatoolkit=11.3 -c pytorch -y"

# Set the working directory
WORKDIR /workspace

# Set the default command to open a bash shell in the Conda environment
CMD ["/bin/bash", "-c", "source /opt/conda/bin/activate pytorch_env && bash"]
```

## Building and Running the Container

### Build the Docker Image
```bash
# Navigate to the directory containing your Dockerfile
cd /path/to/your/dockerfile

# Build the Docker image (replace 'pytorch-conda' with your preferred image name)
docker build -t pytorch-conda .
```

### Run the Container
```bash
# Run the container interactively
docker run -it pytorch-conda

# Run with volume mounting to share files between host and container
docker run -it -v $(pwd):/workspace pytorch-conda

# Run with GPU support (if you have NVIDIA GPU and nvidia-docker installed)
docker run --gpus all -it -v $(pwd):/workspace pytorch-conda
```

## Distributing the Docker Environment

### Method 1: Docker Hub (Recommended)

Docker Hub is the most common way to share Docker images publicly or privately.

#### Push to Docker Hub
```bash
# Tag your image with your Docker Hub username
docker tag pytorch-conda yourusername/pytorch-conda:latest

# Login to Docker Hub
docker login

# Push the image to Docker Hub
docker push yourusername/pytorch-conda:latest
```

#### Pull from Docker Hub
Others can then pull and use your image:
```bash
# Pull the image
docker pull yourusername/pytorch-conda:latest

# Run the container
docker run -it yourusername/pytorch-conda:latest
```

### Method 2: Export/Import Docker Image

For offline distribution or when Docker Hub isn't suitable:

#### Export Image
```bash
# Save the image to a tar file
docker save -o pytorch-conda.tar pytorch-conda:latest
```

#### Import Image
```bash
# Load the image from tar file
docker load -i pytorch-conda.tar

# Run the container
docker run -it pytorch-conda:latest
```

### Method 3: Share the Dockerfile

The simplest method is to share just the Dockerfile:

1. **Create a repository** (GitHub, GitLab, etc.) containing your Dockerfile
2. **Add a README** with build and run instructions
3. **Others can clone and build** the image themselves:
   ```bash
   git clone https://github.com/yourusername/pytorch-docker.git
   cd pytorch-docker
   docker build -t pytorch-conda .
   docker run -it pytorch-conda
   ```

### Method 4: Private Container Registry

For enterprise environments:

1. **Set up a private registry** (AWS ECR, Azure Container Registry, Google Container Registry, or self-hosted)
2. **Tag and push** your image to the private registry
3. **Configure access permissions** for team members
4. **Pull images** using authenticated access

## Best Practices

- **Use .dockerignore** to exclude unnecessary files from the build context
- **Keep images small** by using multi-stage builds and removing unnecessary packages
- **Pin versions** of base images and packages for reproducibility
- **Use non-root users** for security when possible
- **Document your Dockerfile** with comments explaining each step
- **Tag images** with meaningful version numbers
- **Test images** on different platforms before distributing

## Verification

To verify that PyTorch is correctly installed in your container:

```bash
# Run Python in the container and test PyTorch
docker run -it pytorch-conda python -c "import torch; print(f'PyTorch version: {torch.__version__}'); print(f'CUDA available: {torch.cuda.is_available()}')"
```

This tutorial provides a complete workflow for creating, building, and distributing Docker containers with Conda and PyTorch environments, ensuring reproducible development environments across different systems and teams.
