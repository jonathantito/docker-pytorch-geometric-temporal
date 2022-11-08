# Pytorch geometric temporal docker image
## 1) Image of the fork available to use on:

[Docker hub image](https://hub.docker.com/repository/docker/jonathaneduardotitoontaneda/docker-pytorch-geometric-temporal)


## 2) Steps to pubish your own docker image as the one in 1)

### Dockerfile content for this fork:

```
FROM anibali/pytorch:1.11.0-cuda11.5-ubuntu20.04
RUN pip install torch-scatter -f https://data.pyg.org/whl/torch-1.11.0+cu115.html
RUN pip install torch-sparse -f https://data.pyg.org/whl/torch-1.11.0+cu115.html
RUN pip install torch-geometric
RUN pip install torch-cluster -f https://data.pyg.org/whl/torch-1.11.0+cu115.html
RUN pip install torch-spline-conv -f https://data.pyg.org/whl/torch-1.11.0+cu115.html
RUN pip install torch-geometric-temporal
```
#### Build docker image:

```
docker build . -t "DOCKER_HUB_USER/DOCKER_HUB_REPO_NAME"
```

#### Tag docker image builded:

```
docker image ls
```

Copy the IMAGE_ID and run:

```
docker tag IMAGE_ID DOCKER_HUB_USER/DOCKER_HUB_REPO_NAME:torch1.11.0-cuda11.5
```
#### Run docker image in a docker container:

```
docker run --rm -it --init --gpus=all --ipc=host --volume="$PWD:/app" DOCKER_HUB_USER/DOCKER_HUB_REPO_NAME:torch1.11.0-cuda11.5 bash
```
#### Publish on docker hub



As `--rm` option delete the container when the terminal is closed, you must open other terminal and execute:

Login to docker hub using `docker login`


Copy the CONTAINER_ID obtained with:

```
docker ps
```

and run:

```
docker commit CONTAINER_ID DOCKER_HUB_USER/DOCKER_HUB_REPO_NAME:torch1.11.0-cuda11.5

docker push DOCKER_HUB_USER/DOCKER_HUB_REPO_NAME:torch1.11.0-cuda11.5

```

## PyTorch Docker image (We use: anibali/pytorch:1.11.0-cuda11.5-ubuntu20.04)

[![Docker Automated build](https://img.shields.io/docker/automated/anibali/pytorch)](https://hub.docker.com/r/anibali/pytorch/)
[![Docker Automated build](https://img.shields.io/docker/image-size/anibali/pytorch/latest)](https://hub.docker.com/r/anibali/pytorch/)
[![Docker Automated build](https://img.shields.io/docker/pulls/anibali/pytorch)](https://hub.docker.com/r/anibali/pytorch/)
[![Docker Automated build](https://img.shields.io/docker/v/anibali/pytorch/latest)](https://hub.docker.com/r/anibali/pytorch/)

Ubuntu + PyTorch + CUDA (optional)


### Requirements

In order to use this image you must have Docker Engine installed. Instructions
for setting up Docker Engine are
[available on the Docker website](https://docs.docker.com/engine/installation/).

#### CUDA requirements

If you have a CUDA-compatible NVIDIA graphics card, you can use a CUDA-enabled
version of the PyTorch image to enable hardware acceleration. I have only
tested this in Ubuntu Linux.

Firstly, ensure that you install the appropriate NVIDIA drivers. On Ubuntu,
I've found that the easiest way of ensuring that you have the right version
of the drivers set up is by installing a version of CUDA _at least as new as
the image you intend to use_ via
[the official NVIDIA CUDA download page](https://developer.nvidia.com/cuda-downloads).
As an example, if you intend on using the `cuda-10.1` image then setting up
CUDA 10.1 or CUDA 10.2 should ensure that you have the correct graphics drivers.

You will also need to install the NVIDIA Container Toolkit to enable GPU device
access within Docker containers. This can be found at
[NVIDIA/nvidia-docker](https://github.com/NVIDIA/nvidia-docker).


### Prebuilt images

Prebuilt images are available on Docker Hub under the name
[anibali/pytorch](https://hub.docker.com/r/anibali/pytorch/).

For example, you can pull an image with PyTorch 1.8.1 and CUDA 11.1 using:

```bash
$ docker pull anibali/pytorch:1.8.1-cuda11.1
```


### Usage

#### Running PyTorch scripts

It is possible to run PyTorch programs inside a container using the
`python3` command. For example, if you are within a directory containing
some PyTorch project with entrypoint `main.py`, you could run it with
the following command:

```sh
docker run --rm -it --init \
  --gpus=all \
  --ipc=host \
  --user="$(id -u):$(id -g)" \
  --volume="$PWD:/app" \
  anibali/pytorch python3 main.py
```

Here's a description of the Docker command-line options shown above:

* `--gpus=all`: Required if using CUDA, optional otherwise. Passes the
  graphics cards from the host to the container. You can also more precisely
  control which graphics cards are exposed using this option (see documentation
  at https://github.com/NVIDIA/nvidia-docker).
* `--ipc=host`: Required if using multiprocessing, as explained at
  https://github.com/pytorch/pytorch#docker-image.
* `--user="$(id -u):$(id -g)"`: Sets the user inside the container to match your
  user and group ID. Optional, but is useful for writing files with correct
  ownership.
* `--volume="$PWD:/app"`: Mounts the current working directory into the container.
  The default working directory inside the container is `/app`. Optional.

#### Running graphical applications

If you are running on a Linux host, you can get code running inside the Docker
container to display graphics using the host X server (this allows you to use
OpenCV's imshow, for example). Here we describe a quick-and-dirty (but INSECURE)
way of doing this. For a more comprehensive guide on GUIs and Docker check out
http://wiki.ros.org/docker/Tutorials/GUI.

On the host run:

```sh
sudo xhost +local:root
```

You can revoke these access permissions later with `sudo xhost -local:root`.
Now when you run a container make sure you add the options `-e "DISPLAY"` and
`--volume="/tmp/.X11-unix:/tmp/.X11-unix:rw"`. This will provide the container
with your X11 socket for communication and your display ID. Here's an
example:

```sh
docker run --rm -it --init \
  --gpus=all \
  -e "DISPLAY" --volume="/tmp/.X11-unix:/tmp/.X11-unix:rw" \
  anibali/pytorch python3 -c "import tkinter; tkinter.Tk().mainloop()"
```

#### Deriving your own images

The recommended way of adding additional dependencies to an image is to create
your own Dockerfile using one of the PyTorch images from this project as a base.

For example, let's say that you require OpenCV and wish to work with PyTorch
1.8.1. You can create your own Dockerfile using
`anibali/pytorch:1.8.1-cuda11.1-ubuntu20.04` as the base image and install
OpenCV using additional build steps:

```dockerfile
FROM anibali/pytorch:1.8.1-cuda11.1-ubuntu20.04

# Set up time zone.
ENV TZ=UTC
RUN sudo ln -snf /usr/share/zoneinfo/$TZ /etc/localtime

# Install system libraries required by OpenCV.
RUN sudo apt-get update \
 && sudo apt-get install -y libgl1-mesa-glx libgtk2.0-0 libsm6 libxext6 \
 && sudo rm -rf /var/lib/apt/lists/*

# Install OpenCV from PyPI.
RUN pip install opencv-python==4.5.1.48
```


### Development and contributing

The Dockerfiles in the `dockerfiles/` directory are automatically generated by
the `manager.py` script using details in `images.yml` and the templates in
`templates/`.

Here's an example workflow illustrating how to create a new Dockerfile.

1. (Optional) Create a new template file in `templates/` if none of the existing
   ones are appropriate.
2. Create a new entry in `images.yml` (see the existing entries for examples).
3. Generate the Dockerfile by running `python manager.py`. A new directory
   containing the Dockerfile will be created in `dockerfiles/`.
4. Build the generated Dockerfile and test that it works. You can stop here if
   you are creating an image for your own use.
5. (Optional) Submit a PR if you think that your new image might be useful for
   others, and it will be considered for publication.
