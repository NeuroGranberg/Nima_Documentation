Let's walk through the Dockerfile and the auxiliary files step by step, explaining each section and how the additional scripts and files complement the setup. The Dockerfile is designed to build a comprehensive computational environment, integrating various tools and libraries for scientific computing and data analysis.

### Dockerfile Overview [[source]](https://github.com/NeuroGranberg/ng-docker-container/blob/538755f16fc2098287184325d862abc55d2d7b38/Dockerfile#L1C1-L7C10)

```dockerfile
ARG BASE_IMAGE="should be specified with --build-arg"

FROM $BASE_IMAGE

ENV SHELL=/bin/bash

USER root
```
**Explanation:** The Dockerfile starts by specifying a base image through a build argument, allowing users to choose different versions of the base image, such as those with specific CUDA versions for GPU acceleration. So it would be fairly easy to change cuda version by choosing right base image. It sets the default shell to bash and switches to the root user for installing packages and performing configuration.

!!! WARNING root user

    I find out that the root user is required for several tasks, such as configuring the timezone and installing certain packages. However, not every package should be installed with root privileges. This is because the final user may encounter issues when trying to use packages within the container, like fsl and cudimot, without root access.

### Setting Up the Environment [[source]](https://github.com/NeuroGranberg/ng-docker-container/blob/538755f16fc2098287184325d862abc55d2d7b38/Dockerfile#L9C1-L52C43)

```dockerfile
# Add timezone info
ENV TZ=Europe/Stockholm
RUN ln -snf /usr/share/zoneinfo/$TZ /etc/localtime && echo $TZ > /etc/timezone

# apt installs
...
```
**Explanation:** This section configures the timezone and installs necessary packages via `apt-get`. It's crucial for ensuring that the container has all the required system dependencies for the tools and libraries you plan to use. Dependencies of Microstructure Modeling including `Microstructure Optimization Toolbox (MOT)` and `Microstructure Diffusion Toolbox (MDT)` should be installed here.

### Configuring OpenCL [[source]](https://github.com/NeuroGranberg/ng-docker-container/blob/538755f16fc2098287184325d862abc55d2d7b38/Dockerfile#L66C1-L69C47)

```dockerfile
RUN mkdir -p /etc/OpenCL/vendors && \
    echo "libnvidia-opencl.so.1" > /etc/OpenCL/vendors/nvidia.icd
ENV NVIDIA_VISIBLE_DEVICES all
ENV NVIDIA_DRIVER_CAPABILITIES compute,utility
```
**Explanation:** These commands set up OpenCL by pointing to the NVIDIA OpenCL drivers, allowing applications to use GPU acceleration. It configures the container to use all available NVIDIA GPUs with the necessary driver capabilities.

!!! WARNING OpenCL

    Without correctly specifying OpenCL (using the command `echo "libnvidia-opencl.so.1" > /etc/OpenCL/vendors/nvidia.icd`), microstructure modeling tools such as MDT and MOT will not be able to detect GPUs, even if they are passed through to the container and CUDA is available.

### Customizing the Shell [[source]](https://github.com/NeuroGranberg/ng-docker-container/blob/538755f16fc2098287184325d862abc55d2d7b38/Dockerfile#L72C1-L73C34)

```dockerfile
COPY bash.bashrc /etc/bash.bashrc
```
**Explanation:** This is just optional. The custom `bash.bashrc` file is copied to configure the shell environment. This script sets up a custom prompt, enables color support for certain commands, and displays a welcome message or warning about running the container as root.

### Managing Python Packages [[source]](https://github.com/NeuroGranberg/ng-docker-container/blob/538755f16fc2098287184325d862abc55d2d7b38/Dockerfile#L75C1-L81C24)

```dockerfile
COPY requirements.txt .
RUN python3 -m pip install --no-cache-dir pip --upgrade
RUN python3 -m pip install --no-cache-dir -r requirements.txt
...
```
**Explanation:** This part handles Python dependencies. It copies a `requirements.txt` file into the container and uses pip to install these packages. This file lists the Python packages needed for your projects, including direct installations from repositories.

!!! INFO

    The `--no-cache-dir` option prevents pip from saving downloaded packages, reducing image size.

This section of the Dockerfile is dedicated to installing FreeSurfer, a software package for the analysis and visualization of structural and functional neuroimaging data from cross-sectional or longitudinal studies. It's particularly widely used in the study of the brain. Let's break down this section step by step:

### Installation of FreeSurfer [[source]](https://github.com/NeuroGranberg/ng-docker-container/blob/538755f16fc2098287184325d862abc55d2d7b38/Dockerfile#L83C1-L98C47)

```dockerfile
RUN wget https://surfer.nmr.mgh.harvard.edu/pub/dist/freesurfer/7.4.1/freesurfer_ubuntu22-7.4.1_amd64.deb && \
    apt-get install -y --no-install-recommends ./freesurfer_ubuntu22-7.4.1_amd64.deb && \
    rm freesurfer_ubuntu22-7.4.1_amd64.deb

# COPY freesurfer_ubuntu22-7.4.1_amd64.deb .
# RUN  apt-get install -y --no-install-recommends ./freesurfer_ubuntu22-7.4.1_amd64.deb && \
#      rm freesurfer_ubuntu22-7.4.1_amd64.deb
```

1. **Download the FreeSurfer Package:** The `wget` command is used to download the FreeSurfer Debian package (`*.deb`) for version 7.4.1, specifically built for Ubuntu 22.04 (indicated by `ubuntu22` in the filename).

2. **Install the Package:** The `apt-get install` command installs the downloaded `.deb` package. The `--no-install-recommends` option is used to minimize the installation of unnecessary packages, keeping the Docker image size as small as possible.

3. **Cleanup:** After installation, the `.deb` file is no longer needed and is removed with `rm` to save space in the final Docker image.

!!! INFO 

    The commented-out section is an alternative approach where the `.deb` file could be added to the Docker build context manually (using `COPY`) and then installed. This method would be useful if you have a pre-downloaded package or if you want to avoid downloading the package each time you build the Docker image. However, this approach is not used in the final Dockerfile.

#### Setting Environment Variables and Configuring FreeSurfer

```dockerfile
ENV FREESURFER_HOME=/usr/local/freesurfer/7.4.1
COPY license.txt $FREESURFER_HOME
ENV FS_LICENSE=$FREESURFER_HOME/license.txt
ENV SUBJECTS_DIR=$FREESURFER_HOME/subjects
ENV FUNCTIONALS_DIR=$FREESURFER_HOME/sessions
ENV PATH=$FREESURFER_HOME/bin:$PATH
RUN source $FREESURFER_HOME/SetUpFreeSurfer.sh
```

1. **Setting the `FREESURFER_HOME` Environment Variable:** This variable specifies the installation directory of FreeSurfer, which is necessary for its scripts and binaries to function correctly.

2. **Copying the License File:** The `COPY` command places the `license.txt` file into the FreeSurfer home directory. This license file is required to use FreeSurfer and must be obtained by registering on the FreeSurfer website. Without this file, FreeSurfer will not run.

3. **Setting Other Environment Variables:** 
   - `FS_LICENSE` points to the location of the FreeSurfer license file.
   - `SUBJECTS_DIR` and `FUNCTIONALS_DIR` are set to default directories used by FreeSurfer for storing subjects' data and functional analysis sessions, respectively.
   - The `PATH` environment variable is updated to include the FreeSurfer binaries, allowing these tools to be called directly from the command line.

4. **Sourcing the SetUpFreeSurfer.sh Script:** This script sets up the FreeSurfer environment for the current session. It initializes various FreeSurfer environment variables and configurations. The `source` command is used to execute the script in the current shell, ensuring that the Docker container is properly configured to run FreeSurfer commands.


These sections of the Dockerfile are focused on user configuration, installing FSL (FMRIB Software Library), and setting up cuDIMOT, which is a part of the FSL for diffusion MRI analysis. Let's break them down for a clearer understanding:

### User Configuration [[source]](https://github.com/NeuroGranberg/ng-docker-container/blob/538755f16fc2098287184325d862abc55d2d7b38/Dockerfile#L100C1-L109C21)

```dockerfile
# User name is hardcoded to jovyan for compatibility purposes
COPY setup_user.sh /tmp/setup_user.sh
RUN chmod +x /tmp/setup_user.sh && /tmp/setup_user.sh && rm /tmp/setup_user.sh

# Making sure jovyan can install in opt
RUN chown jovyan /opt/

# Fix user
USER jovyan
COPY entrypoint.sh /
```

1. **Setting Up User `jovyan`:** The `setup_user.sh` script is copied to the container, made executable, and then executed. This script ensures that the `jovyan` user is created or updated with the correct UID and necessary permissions. It's a common practice in Jupyter-based Docker images to use `jovyan` as the default user for compatibility with JupyterHub and similar ecosystems.

2. **Permissions for `/opt/`:** The `chown` command changes the ownership of the `/opt/` directory to `jovyan`, ensuring this user has permissions to install or modify software in this directory, which is a common place for installing additional software packages.

3. **Switching to `jovyan` User:** After setting up the user and adjusting permissions, the Dockerfile switches to the `jovyan` user for all subsequent commands. This is a security best practice, as running software as a non-root user reduces the risk of security vulnerabilities.

4. **Preparing Entry Point:** The `entrypoint.sh` script is copied to the root of the container. This script is executed when the container starts and typically contains commands to set up the environment or start services, such as a Jupyter notebook server.

### Installing FSL [[source]](https://github.com/NeuroGranberg/ng-docker-container/blob/538755f16fc2098287184325d862abc55d2d7b38/Dockerfile#L111C1-L114C23)

```dockerfile
COPY fslinstaller.py .
RUN echo | python3 fslinstaller.py --skip_registration
RUN rm fslinstaller.py
```

This part deals with the installation of FSL, a comprehensive library of analysis tools for FMRI, MRI, and DTI brain imaging data. 

1. **Copying and Running the Installer:** The `fslinstaller.py` script is copied into the container and executed. The `echo |` part is a trick to simulate pressing Enter during the installation process, which might be necessary if the installer prompts for user input. The `--skip_registration` option bypasses any registration steps that may be required, streamlining the installation process.

2. **Cleanup:** After installation, the installer script is removed to keep the container clean and reduce its size.


### Setting Up cuDIMOT [[source]](https://github.com/NeuroGranberg/ng-docker-container/blob/538755f16fc2098287184325d862abc55d2d7b38/Dockerfile#L117C1-L125C37)

```dockerfile
ENV CUDIMOT=/opt/CUDIMOT
ENV SGE_ROOT=''
RUN mkdir -p ${CUDIMOT}/bin && \
    wget -qO /tmp/cudimot.zip http://users.fmrib.ox.ac.uk/~moisesf/cudimot/cudimot.zip && unzip -o -d /opt /tmp/cudimot.zip && \
    wget -qO /tmp/NODDI_Watson.zip http://users.fmrib.ox.ac.uk/~moisesf/cudimot/NODDI_Watson/CUDA_10.2/NODDI_Watson.zip && unzip -o -d /tmp /tmp/NODDI_Watson.zip && \
    cp -r /tmp/bin/* ${CUDIMOT}/bin/ && \
    wget -qO /tmp/NODDI_Bingham.zip http://users.fmrib.ox.ac.uk/~moisesf/cudimot/NODDI_Bingham/CUDA_10.2/NODDI_Bingham.zip && unzip -o -d /tmp /tmp/NODDI_Bingham.zip && \
    cp -r /tmp/bin/* ${CUDIMOT}/bin/
```

1. **Environment Setup:** Sets up the `CUDIMOT` environment variable to point to the installation directory of cuDIMOT and clears the `SGE_ROOT` variable, likely as a configuration step for cuDIMOT or related tools.

2. **Downloading and Installing cuDIMOT:** This process involves downloading the cuDIMOT package and additional models (NODDI_Watson and NODDI_Bingham) from specified URLs. These are then unzipped into the designated directory (`/opt/CUDIMOT`), with binary files moved to the `bin` directory under `CUDIMOT`.

This setup enables the container to use cuDIMOT for diffusion MRI analysis, particularly with CUDA 10.2 support for GPU acceleration, enhancing the container's capabilities for neuroimaging analysis.

!!! WARNING

    I discovered that cuDIMOT and FSL should be installed without root privileges because they interfere with Jupyter and other environments, causing them not to function properly in the final container environment. This is the reason why the installation process is now carried out after switching to the `jovyan` user account.

### Auxiliary Files and Scripts

**bash.bashrc**: Customizes the shell environment for users when they start a terminal session within the container. It sets a custom prompt, enables color for `grep` and `ls`, and displays informative messages about running as root or a regular user. This file can be modified to change the shell appearance or behavior.

bash.bashrc [[source]](https://github.com/NeuroGranberg/ng-docker-container/blob/main/bash.bashrc)
```bash
export PS1="\[\e]0;\u@\h: \w\a\]${debian_chroot:+($debian_chroot)}\[\033[01;32m\]\u@ph-notebook\[\e[m\]:\[\033[01;34m\]\w\[\033[00m\]\$ "
export TERM=xterm-256color
alias grep="grep --color=auto"
alias ls="ls --color=auto"

echo -e "\e[1;31m"
cat << "TF"
    _   __                      ______                 __                   
   / | / /__  __  ___________  / ____/________ _____  / /_  ___  _________ _
  /  |/ / _ \/ / / / ___/ __ \/ / __/ ___/ __ `/ __ \/ __ \/ _ \/ ___/ __ `/
 / /|  /  __/ /_/ / /  / /_/ / /_/ / /  / /_/ / / / / /_/ /  __/ /  / /_/ / 
/_/ |_/\___/\__,_/_/   \____/\____/_/   \__,_/_/ /_/_.___/\___/_/   \__, /  
                                                                   /____/  


TF
echo -e "\e[0;33m"

if [[ $EUID -eq 0 ]]; then
  cat <<WARN
WARNING: You are running this container as root, which can cause new files in
mounted volumes to be created as the root user on your host machine.

To avoid this, run the container by specifying your user's userid:

$ docker run -u \$(id -u):\$(id -g) args...
WARN
else
  cat <<EXPL
You are running this container as user with ID $(id -u) and group $(id -g),
which should map to the ID and group for your user on the Docker host. Great!
EXPL
fi

# Turn off colors
echo -e "\e[m"
```

**entrypoint.sh**: This script is executed when the container starts. It prepares the user's home directory, creates symlinks to mounted volumes, and starts the Jupyter notebook server. Modifications can include additional startup tasks or changes to how Jupyter is launched. For example `--NotebookApp.default_url="/lab"` can be `/tree` instead of `/lab` to just head to the JL file explorer. However you can switch to either in the browser address bar.

entrypoint.sh [[source]](https://github.com/NeuroGranberg/ng-docker-container/blob/main/entrypoint.sh)
```bash
#!/bin/bash

echo Preparing content for user home...

# create symlink to /mnt
if [ ! -e /home/jovyan/mnt ]; then
  ln -s /mnt /home/jovyan/
fi

# create symlink to /media
if [ ! -e /home/jovyan/media ]; then
  ln -s /media /home/jovyan/
fi

# Start jupyter service
jupyter notebook --notebook-dir=/home/jovyan \
                 --ip 0.0.0.0 \
                 --no-browser \
                 --allow-root \
                 --NotebookApp.password="$NOTEBOOK_PASSW_SHA1" \
                 --NotebookApp.token="$NOTEBOOK_TOKEN" \
                 --NotebookApp.allow_password_change=True \
                 --NotebookApp.default_url="/lab"
```


**requirements.txt**: Lists the Python packages to be installed in the container. Users can add or remove packages depending on their project needs.


requirements.txt [[source]](https://github.com/NeuroGranberg/ng-docker-container/blob/main/requirements.txt)
```
polars
pandas
mot
mdt
PyQt5
dmri-amico
pybids
git+https://github.com/snakemake/snakemake
```

**setup_user.sh**: Ensures that the `jovyan` user exists with the correct UID and fixes file permissions. It's crucial for managing user access and permissions within the container. This script can be adjusted to change the default user settings or permissions.

setup_user.sh [[source]](https://github.com/NeuroGranberg/ng-docker-container/blob/main/setup_user.sh)
```bash
#!/bin/bash

# Function to update the UID of the 'jovyan' user and fix file ownership
update_uid_and_fix_permissions() {
    # Store the old UID
    OLD_UID=$(id -u jovyan)

    # Change the UID of the 'jovyan' user
    usermod -u 1000 jovyan

    # Find files owned by the old UID and change their ownership to the new UID
    find / -user $OLD_UID -exec chown -h jovyan {} \;
}

# Check if the 'jovyan' user exists
if id "jovyan" &>/dev/null; then
    echo "User 'jovyan' already exists."
    # Check if UID is 1000
    if [ $(id -u jovyan) -ne 1000 ]; then
        echo "User 'jovyan' exists but UID is not 1000, updating UID."
        update_uid_and_fix_permissions
    fi
else
    echo "User 'jovyan' does not exist, creating user."
    adduser --disabled-password --gecos '' --uid 1000 jovyan
fi

# Ensure 'jovyan' is in the 'sudo' group and can use sudo without a password
adduser jovyan sudo
echo '%sudo ALL=(ALL) NOPASSWD:ALL' >> /etc/sudoers
```

**fslinstaller.py**: This is a script to automate the installation of FSL, a library for analyzing FMRI, MRI, and DTI brain imaging data. Users don't interact with this script directly; it's executed within the Dockerfile. To modify its behavior, you might change the installation options or the FSL version being installed.

**license.txt**: Necessary for using FreeSurfer within the container. Users must obtain their license by registering at the FreeSurfer website ([here](https://surfer.nmr.mgh.harvard.edu/registration.html)) and place the `license.txt` file alongside the Dockerfile. This requirement ensures compliance with FreeSurfer's licensing.

!!! INFO 
    Each of these auxiliary files plays a critical role in configuring the environment, managing permissions, and installing necessary software, making the container a powerful tool for scientific computing and analysis.