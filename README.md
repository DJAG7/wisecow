# Wisecow Deployment

## Overview
This repository contains the configuration and deployment setup for the Wisecow server application based on Cowsay and Fortune. It integrates Docker for containerization, GitHub Actions for CI/CD, and Kubernetes for orchestration. It runs locally using act on the development server, but can be integrated with cloud based deployment/ CloudFormation.

This has been configured on Ubuntu, and other OSes could differ based on versions. Please refer prerequisite documentation

## Prerequisites


* **nginx**: Tool for displaying the webapp / website. This code is configured to run and be deployed on nginx only but can be refactored with Apache2. Run ``` sudo apt install nginx ```

* **openssl**: Tool for generating TLS. Also requires a domain or public IP for secure connection. Run ``` sudo apt install openssl ```

* **Docker**: Install Docker from [Docker's official website](https://www.docker.com/get-started).

After running docker, start and initialize docker as a seperate user.

* **Kubernetes Cluster**: You can use Minikube for local testing or any remote Kubernetes cluster. In this project, I have used Minikube and kubectl.

Verify installation of minikube by using minikube status. Do not start minikube as it will be started later on using github action workflows.

* **GitHub Actions**: Set up for CI/CD workflows.

Download Github Client, and authenticate using ``` gh auth login ```. Use a personal access token.

* **act**: Tool for running GitHub Actions locally. Install it from [act's GitHub page](https://github.com/nektos/act). 
You can install act using the following commands

```terminal

curl --proto '=https' --tlsv1.2 -sSf https://raw.githubusercontent.com/nektos/act/master/install.sh | sudo bash
sudo mv ./bin/act /your/repo
```
This moves it from the main installation directory to the repository directory that can be accessed easily by act

## Repository Structure

* `Dockerfile`: Defines the Docker image for the Wisecow application.
* `wisecow-deployment.yaml`: Kubernetes deployment configuration.
* `wisecow-service.yaml`: Kubernetes service configuration.
* `.github/workflows/ci-cd.yml`: GitHub Actions workflow for CI/CD.
* `.env`: Environment file for local testing with `act`. Required if using Docker Hub Sign in

## Docker Setup


There are 2 ways of using Docker to build and run the image. I have built it manually, and uploaded wisecow as a public docker image which can be hosted.

If you are using Docker with Github Actions, skip the following 2 Steps:


### Build the Docker Image

Ensure that you have a valid `Dockerfile` in the root directory of your repository. To build the Docker image, use the following command:

```bash
docker build -t your-dockerhub-username/wisecow:latest .
```

### Push the Docker Image
Once the image is built, push it to DockerHub:

```bash
docker push your-dockerhub-username/wisecow:latest
```

Replace your-dockerhub-username with your DockerHub username.


![image](https://github.com/user-attachments/assets/5e524eef-5c1b-4402-a69c-3c824175b6e1)

If you are accessing this repository, you can use my image from dockerhub djag7/wisecow:latest.

## Kubernetes Configuration and TLS

Generate the TLS Certificate using the cert tool using ``` openssl req -new -x509 -days 365 -key private.key -out certificate.crt ```

This generates both the private key and the TLS certificate


Update the ingress.yaml file with the location of the TLS certificate. Relocate certificates to the same directory as the repository

![image](https://github.com/user-attachments/assets/7119213a-e69c-4a77-b592-aed4e4681a34)


## Github Actions Config

GitHub Actions

As mentioned above, there are two ways to run github actions, either by using a prebuilt docker image or by building a new docker image

If you are building a new docker image, include docker username and password in a .env file and modify actions file to include ``` docker build -t image name ``` and ``` docker push image-name```
.env file should look like this
![image](https://github.com/user-attachments/assets/dc10e16e-e9b0-45e3-a8d4-e10f1d5c6f1a)

copy kubectl config file text and paste it in the .env file located in /wisecow/
![image](https://github.com/user-attachments/assets/4fe275d1-1255-46ca-8dc8-c2eca9f220ae)

in the env file write it this way

```env
KUBECONFIG=" kube config text"
```

To run Github actions navigate to the wisecow directory and run locally

```terminal
act -P ubuntu-latest=ghcr.io/catthehacker/ubuntu:full-latest --env-file .env
```

Choose Medium dependency run, and press enter. It will begin

Env file with docker
```env
DOCKER_USERNAME=your_dockerhub_username
DOCKER_PASSWORD=your_dockerhub_password
KUBECONFIG=/path/to/your/kubeconfig
```

## Nginx Configuration

Run minikube ingress for nginx 

``` minikube addons enable ingress ```

Update minikube as a local host using minikube ip generated from github actions

```sudo echo "$(minikube ip) wisecow.local" >> /etc/hosts```

It can be accessed via the domain

![image](https://github.com/user-attachments/assets/a1176334-f897-4c8c-87c2-4a85bd6fac01)





