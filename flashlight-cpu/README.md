# Flashlight CPU (OpenCL) Docker Image
This Docker image is tailored for running the Flashlight framework on CPU systems with OpenCL support, enabling efficient parallel processing across multiple cores.

## Building the Image
To build the Docker image that leverages OpenCL on CPU, navigate to the directory containing the Dockerfile (labeled for CPU/OpenCL support) and execute the following command:

```sh
    docker build -t flashlight-cpu:latest . 
```

### Proxy Configuration
If your network requires additional proxy configurations (e.g., `HTTPS_PROXY`), you can add them by extending the build command with more build arguments:

```sh
docker build -t flashlight-cpu:latest . --build-arg HTTP_PROXY=http://<PROXY_HOST>:<PROXY_PORT> --build-arg HTTPS_PROXY=https://<PROXY_HOST>:<PROXY_PORT>
```

Replace `<PROXY_HOST>` and `<PROXY_PORT>` with the hostname and port number of your proxy server, respectively.



## Running the Container
Before running the container, ensure your CPU supports OpenCL and the necessary software is installed. Start the container with the following command:

```sh
docker run -it \
           -v <YOUR_LOCAL_PATH>:/app \
           --name flcpu \
           flashlight-cpu:latest 
```


Replace `<YOUR_LOCAL_PATH>` with the path to the directory on your local machine that you want to share with the container.

The `-v` flag mounts your local files into the `/app` directory within the container.

The `--name flcpu` flag assigns the name `flcpu` to the container, which helps in managing and identifying containers.

## Additional Notes
- **OpenCL Compatibility:** 
Confirm that your CPU model and operating system support OpenCL. Common CPUs that support OpenCL include those from Intel and AMD.
- **Software Requirements:**
Ensure that OpenCL runtime and development libraries are installed on your system.
- **Performance Optimization:**
Configure your Flashlight applications to take full advantage of OpenCL capabilities for improved performance.

For comprehensive information on Flashlight and OpenCL, visit the [Flashlight documentation](https://github.com/flashlight/flashlight). If you encounter issues or have questions about this Docker image, consider creating an issue on the project's repository or contacting the maintainers.