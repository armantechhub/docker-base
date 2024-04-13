from ubuntu:20.04

RUN apt update -y

RUN apt install build-essential -y      
RUN apt install g++ -y
RUN apt install cmake -y
RUN apt install python3-pip -y
RUN apt install python3-venv -y
RUN apt install python3-dev -y
RUN apt install python-is-python3 -y
RUN apt install zip -y
RUN apt install unzip -y
RUN apt install tree -y
RUN apt install git -y

RUN pip3 config --user set global.index-url http://repo.farbeen.local/repository/pypi/simple
RUN pip3 config --user set global.trusted-host repo.farbeen.local

RUN pip3 install ruff          
RUN pip3 install pytest        
RUN pip3 install pytest-cov    
RUN pip3 install pytest-xdist  
RUN pip3 install twine         
RUN pip3 install build         
RUN pip3 install wheel         
RUN pip3 install pipx          
RUN pip3 install scikit-build  
RUN pip3 install pybind11      
RUN pip3 install nanobind      
RUN pip3 install black


CMD ["tail", "-f", "/dev/null"]
