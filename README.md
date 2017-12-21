# README

### Starting the app

Prerequisites are that you have Git client installed and have cloned this repo. You also need Docker for Mac installed.

1. Run `docker-compose up`
1. Visit the application at `http://localhost`

Note that you can use the `-d` switch with `docker-compose up` to detatch the running app from the terminal. This can be followed by a log output command so to run the app you can open a terminal and run the following:

`docker-compose up -d && docker-compose logs -f`

## Development

### Add a new Rails resource

Run the `rails` command line via `docker-compose` to execute the commands within the Docker image. Do this by prefixing each command with `docker-compose run --rm webapp`

For example run `docker-compose run --rm webapp bin/rails g scaffold articles title:string body:text` (change the model name and fields as per what is needed)

### Connect a terminal session to the container

Run `docker exec -it webapp bash` which will connect you a permanent terminal session to the container. Then it's possible to run `rails`, `rake` (etc) commands without the prefix (since you are 'inside' the container now).

### Update the Docker Image

We will append the short commit SHA to the remote Docker Image name. See contents of push.sh file for details.

`./push.sh`

### Deploying the application to AWS Production environment 

### TODO: Stay Tuned!

### Setup a new Dockarized Rails Project

NOTE: Accociated tutorial repo is [here](https://github.com/Apress/deploying-rails-w-docker/tree/master/webapp)

What follows are brief notes on how I dockarised this Rails application and pushed it to Docker Hub.

I created the application myself so I decided to install Rails on my local computer and create the application there. 

`rails new --skip-bundle --api --database postgresql`

Mote we passed `--skip-bundle` which means that the applications gem dependencies have not yet been installed and therefore there is no `Gemfile.lock`. The reason for doing this is so that we can run `bundle install` inside a Docker image like so.

`docker run --rm -v "$PWD":/usr/src/app -w /usr/src/app ruby:2.4.1 bundle install`

This will create our Gemfile.lock file. The remaining files are added and configured manually as follows:

* Add `webapp.conf` 
* Add `rails-env.conf`
* Add `Dockerfile`
* Add `setup.sh`
* Add `docker-compose.yml`
* Add a customer logger in `config\application.rb` which sends logs to STDOUT so that we can use `docker-compose logs -f` (see below for starting the app).
* Add `.dockerignore`
* Add `push.sh` file for building and pushing the Docker image to Docker Hub.

See all the above files in this repo for example content in each. Once all the above files are completed just run `./push.sh` to build and push the Docker image to Docker Hub (log into the `docker` cli tool first).

### Setup Kubernetes in a new project

The following is a high level step by step guideline to setting up Kubernetes for a new project as well as testing the deployment using minkkube

* Create directories `kube/deployments` & `kube/jobs`
* Create the deployment files  (see files in this repo for examples)
* Create the job files (see files in this repo for examples)
* Install [minikube](https://github.com/kubernetes/minikube) to test the deployment locally. Make sure that your deployment and job files use the development environment for this testing.
* Install [kubectl](https://kubernetes.io/docs/getting-started-guides/minikube/#download-kubectl)
* Install VirtualBox
* Start minikube by running `minikube start`
* Run `minikube dashboard`
* Run `kubectl create -f kube/deployments/postgres-deployment.yaml`
* Run `kubectl create -f kube/jobs/setup-job.yaml`
* Wait for the job to complete by checking the logs in the dashboard
* Run `kubectl create -f kube/deployments/webapp-deployment.yaml`
* Run `kubectl get pods` which will list all the running pods (in this project it will show one db and three webapp pods)
* Run `minikube service webapp` (this will open up the application with load balancing already configured and working accross the three application pods )
* Run `kubectl describe service webapp` to view details about the service created

### AWS CLI Tips

To query the API using the AWS CLI use the --query switch. For example, to return the instance id's of all EC2 instances, run the following:

`aws ec2 describe-instances --query='Reservations[*].Instances[*].InstanceId'`

Use filters to fetch details about a specific subset of data. For example, fetch the Subnets for a specifiv VPC as follows:

`aws ec2 describe-subnets --filters "Name=vpc-id,Values=vpc-471eff20" --query="Subnets[*].SubnetId"`