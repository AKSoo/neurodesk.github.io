---
title: "Linux"
linkTitle: "Linux"
weight: 1
description: >
  Install neurodesktop on Linux
---

## Minimum System Requirements
1. At least 3GB free space for neurodesktop base image
2. Docker requirements. Details found under https://docs.docker.com/get-docker/

## Quickstart
### 1. Install Docker
Install Docker from here: https://docs.docker.com/get-docker/. Additional information available [below](#installing-docker)

### 2. Run Neurodesktop
Before the first run, create a local folder where the downloaded applications will be stored, e.g. `~/neurodesktop-storage`

Then use one of the following options to run Neurodesktop:

#### Option 1: NeuroDesktop.run
Download and run the following executable
https://github.com/NeuroDesk/neurodesktop/raw/main/Linux_run_Neurodesk/NeuroDesktop.run

#### Option 2: Using Terminal
1. Open a terminal, and type the folowing command to automatically download the neurodesktop container and run it (Mac, Windows, Linux commands listed below) 

<pre class="language-shell command-line" data-prompt="$" data-output="2-6">
<code>sudo docker run \
  --shm-size=1gb -it --privileged --name neurodesktop \
  -v ~/neurodesktop-storage:/neurodesktop-storage \
  -e HOST_UID="$(id -u)" -e HOST_GID="$(id -g)" \
  -p 8080:8080 -h neurodesktop-{{< params/neurodesktop/version >}} \
  vnmd/neurodesktop:{{< params/neurodesktop/version >}}</code>
</pre>
<!-- neurodesktop version found in neurodesk.github.io/data/neurodesktop.toml -->

{{< alert >}}
If you get errors in neurodesktop then check if the ~/neurodesktop-storage directory is writable to all users, otherwise run `chmod a+rwx ~/neurodesktop-storage`
{{< /alert >}}

3. Once neurodesktop is downloaded i.e. `guacd[77]: INFO:        Listening on host 127.0.0.1, port 4822` is displayed in terminal, open a browser and go to:
```
http://localhost:8080/#/?username=user&password=password
```
or open a VNC Client and connect to port 5901 (for this -p 5901:5901 has to be added to the docker call)

4. neurodesktop is ready to use!
- User is `user`
- Password is `password`

## Stopping neurodesktop:
When done processing your data it is important to stop and remove the container - otherwise the next start or container update will give an error ("... The container name "/neurodesktop" is already in use...")
1. Click on the terminal from which you ran neurodesktop

2. Press `Ctrl-C`

3. Run:
<pre class="language-shell command-line" data-prompt="$">
<code>docker stop neurodesktop</code>
</pre>

4. Run:
<pre class="language-shell command-line" data-prompt="$">
<code>docker rm neurodesktop</code>
</pre>

## Installing Docker

For general installation instructions, refer to https://docs.docker.com/get-docker/

### RHEL/CentOS (yum-based)
Refer to https://docs.docker.com/engine/install/centos/

One example to install docker in a yum-based distribution could look like this:
<pre class="language-shell command-line" data-prompt="$">
<code>sudo dnf install -y yum-utils 
sudo yum-config-manager --add-repo https://download.docker.com/linux/centos/docker-ce.repo
sudo dnf install docker-ce docker-ce-cli containerd.io
sudo systemctl enable docker
sudo systemctl start docker
sudo docker version
sudo docker info
sudo groupadd docker
sudo usermod -aG docker $USER
sudo chown root:docker /var/run/docker.sock
newgrp docker</code>
</pre>

### Ubuntu/Debian (apt-based)
Refer to https://docs.docker.com/engine/install/ubuntu/

One example to install docker in a apt-based distribution could look like this:
<pre class="language-shell command-line" data-prompt="$" data-output="5">
<code>sudo apt-get update
sudo apt-get install apt-transport-https ca-certificates curl gnupg lsb-release
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /usr/share/keyrings/docker-archive-keyring.gpg
echo "deb [arch=$(dpkg --print-architecture) signed-by=/usr/share/keyrings/docker-archive-keyring.gpg] https://download.docker.com/linux/ubuntu \
  $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
sudo apt-get update
sudo apt-get install docker-ce docker-ce-cli containerd.io</code>
</pre>

## GPU support

### RHEL/CentOS (yum-based)
<pre class="language-shell command-line" data-prompt="$" data-output="4-9">
<code>sudo yum install nvidia-container-toolkit -y</code>
</pre>
### Running neurodesktop container with GPU
<pre class="language-shell command-line" data-prompt="$" data-output="2-9">
<code>sudo docker run \
  --shm-size=1gb -it --privileged --name neurodesktop \
  -v ~/neurodesktop-storage:/neurodesktop-storage \
  -e HOST_UID="$(id -u)" -e HOST_GID="$(id -g)" \
  -e NVIDIA_VISIBLE_DEVICES=all \
  -e NVIDIA_DISABLE_REQUIRE=1 \
  -p 8080:8080 -h neurodesktop-{{< params/neurodesktop/version >}} \
  vnmd/neurodesktop:{{< params/neurodesktop/version >}}</code>
</pre>

<!-- neurodesktop version found in neurodesk.github.io/data/neurodesktop.toml -->

Then inside the neurodesktop container run:
<pre class="language-shell command-line" data-prompt="$">
<code>
sudo apt update
sudo apt install libcudart10.1
</code>
</pre>

For a GPU with Nvidia driver Version > 495.29.05:
<pre class="language-shell command-line" data-prompt="$">
<code>
wget https://developer.download.nvidia.com/compute/cuda/11.5.0/local_installers/cuda_11.5.0_495.29.05_linux.run
sudo sh ./cuda_11.5.0_495.29.05_linux.run
</code>
</pre>

#### Running tensorflow (w/ GPU)
##### Using tensorflow (python)
<pre class="language-shell command-line" data-prompt="$">
<code>
conda install tensorflow-gpu
python</code>
</pre>
<pre class="language-python command-line" data-prompt=">>>">
<code>import tensorflow as tf
print("Num GPUs Available: ", len(tf.config.list_physical_devices('GPU')))</code>
</pre>

![image](https://user-images.githubusercontent.com/4021595/135446560-d135f6ce-b699-4e46-8534-b72b4d9f2d41.png)
##### Using tensorflow (singularity container in neurodesktop)
<pre class="language-shell command-line" data-prompt="$">
<code>singularity pull docker://tensorflow/tensorflow:latest-gpu
singularity run --nv tensorflow_latest-gpu.sif
python</code>
</pre>
<pre class="language-python command-line" data-prompt=">>>">
<code>import tensorflow as tf
print("Num GPUs Available: ", len(tf.config.list_physical_devices('GPU')))</code>
</pre>

![image](https://user-images.githubusercontent.com/4021595/135449288-6c3e9bbd-fe5f-4f43-aa4a-8a798ba629e6.png)
