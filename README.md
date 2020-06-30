[![CircleCI](https://circleci.com/gh/thrld/project-ml-microservice-kubernetes.svg?style=svg)](https://circleci.com/gh/thrld/project-ml-microservice-kubernetes)

## Project Overview

In this project, you will apply the skills you have acquired in this course to operationalize a Machine Learning Microservice API. 

You are given a pre-trained, `sklearn` model that has been trained to predict housing prices in Boston according to several features, such as average rooms in a home and data about highway access, teacher-to-pupil ratios, and so on. You can read more about the data, which was initially taken from Kaggle, on [the data source site](https://www.kaggle.com/c/boston-housing). This project tests your ability to operationalize a Python flask app—in a provided file, `app.py`—that serves out predictions (inference) about housing prices through API calls. This project could be extended to any pre-trained machine learning model, such as those for image recognition and data labeling.

### Project Tasks

Your project goal is to operationalize this working, machine learning microservice using [kubernetes](https://kubernetes.io/), which is an open-source system for automating the management of containerized applications. In this project you will:
* Test your project code using linting
* Complete a Dockerfile to containerize this application
* Deploy your containerized application using Docker and make a prediction
* Improve the log statements in the source code for this application
* Configure Kubernetes and create a Kubernetes cluster
* Deploy a container using Kubernetes and make a prediction
* Upload a complete Github repo with CircleCI to indicate that your code has been tested

You can find a detailed [project rubric, here](https://review.udacity.com/#!/rubrics/2576/view).

**The final implementation of the project will showcase your abilities to operationalize production microservices.**

---

## Setup the Environment

* Create a virtualenv and activate it
* Run `make install` to install the necessary dependencies

### Running `app.py`

1. Standalone:  `python app.py`
2. Run in Docker:  `./run_docker.sh`
3. Run in Kubernetes:  `./run_kubernetes.sh`

### Kubernetes Steps

* Setup and Configure Docker locally
* Setup and Configure Kubernetes locally
* Create Flask app in Container
* Run via kubectl

## Own Remarks

### Why I used an EC2 instance of my local Windows

Starting `minikube` on Windows was rather involved. Here is what I did to make it work:

1. Run `systeminfo` in Powershell to check if virtualisation is supported (see [here](https://kubernetes.io/docs/tasks/tools/install-minikube/#before-you-begin)
 for more info).
1. Install `kubectl` on Windows using [this command](https://kubernetes.io/docs/tasks/tools/install-kubectl/#install-kubectl-on-windows)
1. Install `minikube`: Open an elevated Powershell and type `choco install minikube`. Close the CLI session.
1. In theory, a new elevated Powershell session and a simple `minikube start` should do the job. Not in my case, 
though. What helped me was to [set up a new external network switch](https://docs.docker.com/machine/drivers/hyper-v/#2-set-up-a-new-external-network-switch-optional)
1. In an elevated Powershell session, the following now worked for me: 
`minikube start --vm-driver=hyperv --hyperv-virtual-switch="Primary Virtual Switch"`.

However, I only managed to start `minikube` on Windows but not inside WSL2, which still seems to be unsolved problem. Hence, I switched to an EC2 instance.
 
### Setting up the EC2 instance

Here are the steps I took to set everything up.

- Start an EC2 instance (`t2.medium`), leaving all other settings at their defaults. 
- Before launching the machine, I create a new keypair and download the `.pem` file.
- From then on, I work within my Ubuntu shell. To change the file permissions, I have to first copy over the `.pem` file from the Windows `Downloads` directory to the Linux file system. Type: 
```
cd /mnt/c/Users/<your-user>/Downloads
mv <your-key>.pem /tmp
cd /tmp
chmod 400 <your-key>.pem
ssh -i "<your-key>.pem" ubuntu@<your-ec2>.us-east-2.compute.amazonaws.com
```
- [Install `kubectl`](https://kubernetes.io/docs/tasks/tools/install-kubectl/#install-kubectl-on-linux). Type:
```
curl -LO https://storage.googleapis.com/kubernetes-release/release/`curl -s https://storage.googleapis.com/kubernetes-release/release/stable.txt`/bin/linux/amd64/kubectl
chmod +x ./kubectl
sudo mv ./kubectl /usr/local/bin/kubectl
kubectl version --client
which kubectl
```
- Install `Docker` using the [convenience script](https://docs.docker.com/engine/install/ubuntu/#install-using-the-convenience-script). Type:
```
curl -fsSL https://get.docker.com -o get-docker.sh
sudo sh get-docker.sh
sudo usermod -aG docker ubuntu       
```
Log out and back in for this to take effect!

- Install `minikube` [via direct download](https://kubernetes.io/docs/tasks/tools/install-minikube/#install-minikube-via-direct-download). Type:
```
curl -Lo minikube https://storage.googleapis.com/minikube/releases/latest/minikube-linux-amd64 \
  && chmod +x minikube
sudo mkdir -p /usr/local/bin/
sudo install minikube /usr/local/bin/
which minikube
```
- `minikube` with `vm-driver=none` has to be started as the root user, so type: `sudo su -`.
- Install `conntrack` in the root path since `minikube` will throw an error upon startup otherwise: `sudo apt-get install -y conntrack`
- Finally, start up `minikube`: `minikube start --vm-driver=none`
- `kubectl` should now be configured to use `minikube`, so typing `kubectl get nodes` should yield one `master` node with status `Ready`. Also check `kubectl cluster-info` and `minikube status`
- To list available addons for `minikube`, type `minikube addons list`. To enable `dashboard`, type `minikube addons enable dashboard`. Then list the dashboard URL (`minikube dashboard --url`). To make the URL accessible from my own browser, I take note of the port (in my case: `34727`) and open an SSH tunnel, so that this port will be available on port `8081` locally: `ssh -i /tmp/<your-key>.pem -L 8081:localhost:34727 ubuntu@<your-ec2>.us-east-2.compute.amazonaws.com`. Now copy the supplied link, paste it into to your local web browser, and change the port to 8081 (something like: `http://127.0.0.1:8081/api/v1/namespaces/kubernetes-dashboard/services/http:kubernetes-dashboard:/proxy/#/overview?namespace=default`).

> Note that once you interrupt either of the commands (`minikube dashboard --url` or `ssh -i /tmp/<your-key>.pem -L 8081:localhost:34727 ubuntu@<your-ec2>.us-east-2.compute.amazonaws.com`), the dashboard will stop working!

- Now let us make sure we can use `git`. Follow the instructions for creating and registering an SSH key.
- Install `python3-venv`: `sudo apt-get install python3-venv`
- Create the `venv` and activate it:
```
python3 -m venv ~/.devops
source ~/.devops/bin/activate
```
- Install `make`: `sudo apt install make`
- Install pip packages: `make install`
- Install additional required package: `pip install pylint` (or add to `requirements.txt` before running `make install`)
- Install hadolint: `wget -O /bin/hadolint https://github.com/hadolint/hadolint/releases/download/v1.18.0/hadolint-Linux-x86_64 && sudo chmod +x /bin/hadolint`
- Run `make lint`. You will get a score slightly below `10.0`. To fix this, surpress these errors during linting by adapting the `Makefile`: `pylint --disable=R,C,W1203,W1309 app.py`. Running `make lint` again should yield a score of `10.0` now.

### Pushing to DockerHub

After making the required changes and ensuring that everything runs smoothly, we are ready to push our image to DockerHub.
- Log in: `docker login -u <your-user> --p <your-pwd>`
- And push: `./upload_docker.sh`

### Running in k8s

- Install `socat`, otherwise port forwarding won't work: `sudo apt-get -y install socat`
- Type: `./run_kubernetes.sh`

