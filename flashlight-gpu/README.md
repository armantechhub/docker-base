# Flashlight GPU Docker Image

This is a Docker image for the [Flashlight](https://github.com/flashlight/flashlight) framework with GPU support.

## Building the Image
To build the Docker image, navigate to the directory containing the Dockerfile and execute the following command:

```sh
docker build -t flashlight-gpu:latest .
```

### Proxy Configuration
If your network requires additional proxy configurations (e.g., `HTTPS_PROXY`), you can add them by extending the build command with more build arguments:

```sh
docker build -t flashlight-cpu:latest . --build-arg HTTP_PROXY=http://<PROXY_HOST>:<PROXY_PORT> --build-arg HTTPS_PROXY=https://<PROXY_HOST>:<PROXY_PORT>
```

Replace `<PROXY_HOST>` and `<PROXY_PORT>` with the hostname and port number of your proxy server, respectively.

## Running the Container
Before you can run the container, ensure that you have installed the NVIDIA Container Toolkit. You can find installation instructions for your system at the [NVIDIA Container Toolkit Installation Guide](https://docs.nvidia.com/datacenter/cloud-native/container-toolkit/latest/install-guide.html).

To run the container, use the following command:

```sh
docker run -it \
           -v <YOUR_LOCAL_PATH>:/app \
           --name flgpu \
           --runtime=nvidia \
           flashlight-gpu:latest
```
Replace `<YOUR_LOCAL_PATH>` with the path to the directory on your local machine that you want to mount inside the container.

The `-v` flag mounts the local directory to the `/app` directory inside the container, allowing you to access your files within the container environment.

The `--runtime=nvidia` flag ensures that the container has access to the GPU resources provided by the NVIDIA Container Toolkit.

The `--name flgpu` flag sets the name of the container to `flgpu`, which can be useful for managing containers.

Once the container is running, you can interact with it as you would with any other Linux environment, with the added benefit of GPU acceleration for your Flashlight applications.

## Additional Notes
Make sure your system meets the requirements for running GPU-accelerated containers before proceeding. This includes having an NVIDIA GPU and the appropriate drivers installed on your host system.

For more information on Flashlight, refer to the [official documentation](https://github.com/flashlight/flashlight). For questions or issues regarding this Docker image, feel free to open an issue on the repository or contact the maintainers.  