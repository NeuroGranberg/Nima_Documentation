Creating and using a Docker container for neuroimaging analysis involves several steps, including building the Docker image from a Dockerfile and then running the container. Additionally, users can customize the setup by modifying auxiliary files or use a pre-built image with Docker Compose for an easier setup. Here's how to navigate these processes:

## Using the Dockerfile

### Building the Image

1. **Prepare Auxiliary Files:** Ensure you have all the necessary auxiliary files (`bash.bashrc`, `entrypoint.sh`, `requirements.txt`, `setup_user.sh`, `fslinstaller.py`, and your `license.txt` for FreeSurfer) in the same directory as your Dockerfile.
2. **Build Command:** Navigate to the directory containing your Dockerfile and run the following command to build your Docker image, replacing `<your-image-name>` with your chosen name for the image:
   ```bash
   docker build --build-arg BASE_IMAGE=cschranz/gpu-jupyter:v1.6_cuda-12.0_ubuntu-22.04 -t <your-image-name> .
   ```
3. **Running Your Container:** After building the image, you can start a container from it using:
   ```bash
   docker run -it --name <container-name> --gpus all -p 8888:8888 <your-image-name>
   ```
   Adjust the `-p` argument to map any ports you need (e.g., for Jupyter notebooks).

### Modifying Auxiliary Files

- **bash.bashrc:** Customize the shell environment, like changing the command prompt or setting aliases. Modify this file to adjust the look and behavior of the terminal within the container.
- **entrypoint.sh:** Contains commands executed when the container starts. Modify it to add any startup tasks or change how the Jupyter server starts.
- **requirements.txt:** List of Python packages to install. Add or remove packages to suit your project's needs.
- **setup_user.sh:** Manages the `jovyan` user setup and permissions. Modify UID/GID or add additional setup steps as needed.
- **fslinstaller.py:** Script for installing FSL. Normally, this doesn't need modification, but you could adjust installation options if necessary.
- **license.txt:** Your FreeSurfer license file. Ensure it's up-to-date and correctly placed to use FreeSurfer in your container.

## Pre-built Image with Docker Compose

For those preferring to use a pre-built image and simplify the deployment, Docker Compose can manage the container setup using a `docker-compose.yml` file. This can be used for example as a Portainer Stack.

#### docker-compose.yml Explained

```yaml
version: '3.8'

services:
  cuda-env:
    image: nimach/neurog-full-img:1.0
    container_name: neurog-full-toolkit # Change this to include your name
    environment:
      NVIDIA_VISIBLE_DEVICES: all
      NVIDIA_DRIVER_CAPABILITIES: all
    ports:
      - "xxxx:8888"
    volumes:                            # Change this part to include your desired folder on the host
      - /mnt:/mnt
      - /media:/media
      - /home/USERNAME:/home/jovyan/work
    restart: unless-stopped
    deploy:
      resources:
        reservations:
          devices:
          - driver: nvidia
            device_ids: ['0', '1']      # Select gpus that you want to have access to
            capabilities: [gpu]
```

- **Image:** Specifies the Docker image to use. Here, it's using `nimach/neurog-full-img:1.0`, which should be the image built from the Dockerfile or a similar pre-built image.
- **Container Name:** Sets a custom name for your container.
- **Environment:** Configures the container to use all NVIDIA GPUs with full driver capabilities.
- **Ports:** Maps port xxxx on the host to port 8888 inside the container, typically used for Jupyter Notebook/Lab servers.
- **Volumes:** Mounts directories from the host inside the container for data persistence and access to datasets. You can adjust `/home/USERNAME` line to mount your home directory address as needed.
- **Restart Policy:** Configures the container to restart automatically unless manually stopped.
- **GPU Reservation:** Explicitly reserves NVIDIA GPUs for the container, specifying which GPUs to use (`device_ids`) and their capabilities.

### Modifying the docker-compose.yml

- **To change the image version or source:** Update the `image:` line with the new image name or version.
- **Customizing Ports:** You should adjust the `xxxx` in `"xxxx:8888"` mapping to a different ports that does not confilict with other container port for accessing services like Jupyter.
- **Adjusting Volume Mounts:** Modify or add entries under `volumes:` to change or add more host directories mounted inside the container.
- **GPU Configuration:** If you have different GPU requirements, modify `device_ids` or `capabilities` under `resources:` to match your needs based on the hardware resources.

!!! INFO
    Using Docker Compose simplifies running and managing Docker containers, especially for complex configurations like this neuroimaging environment. It allows for easy customization and scaling across different machines and projects.