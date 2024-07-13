docker build -t flashlight-gpu:latest .
docker run -it -v <YOUR_LOCAL_PATH>:/app --name flgpu --runtime=nvidia flashlight-gpu:0.0.2

https://docs.nvidia.com/datacenter/cloud-native/container-toolkit/latest/install-guide.html