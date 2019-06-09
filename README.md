# applinker
For this project, we were given a set of components around which to build a continuous integration pipeline. The end result would be a secure page with which one could register or login to a webpage
## Architecture
1 gateway is dependent on 2 clients and a series of services, 8 in total. The last service in the line of dependency is mongo-service, and this inn turn depends on a mongo database. There are two other service components but they were made redundant in the pipeline. The dependencies are charted out as follows, where dependency is from the bottom up (with implementation to match)
![Network of dependencies](https://github.com/CoryGreen/applinkerdiagram/blob/master/applinker.drawio)
## Requirements
A new project is started, this time on the Google Cloud Platform. On the platform and in the project a Virtual Machine Instance with 2 CPU cores and no less than 4GB memory, and featuring Ubuntu 18.04, is created as the first step. An email account is also required with which to send out activation links for newly registered users. And an image for each service and client should be pushed to DockerHub
## Procedure
After moving the code to source control, a dockerfile is created for each service and client and the gateway, all in the Virtual Machine Instance. All are then built along with the database, and their dependencies put into effect, with a docker-compose.yaml. The docker-compose.yaml also creates the environment variables for the email-service and the authentication-service and publishes the ports for the gateway. The docker-compose.yaml is the implemented:
```
docker-compose build .
docker-compose up -d
docker-compose push
```
Success of the procedure is demonstrated by a corresponding working website. The last line of code above ensures the presence of corresponding images in DockerHub

After source control is updated with these changes, a kubernetes cluster is created in the platform's  Cloud Shell:
```
gcloud container clusters create <cluster-name> --region <region>
gcloud container clusters get-credentials <cluster-name> --region <region>
```
Afterwards, the updated repository is cloned from source control to the Cloud Shell and a service.yaml and deployment.yaml file built for every service and client and the mongo database as required. While each service file sends each service to port 3000 and each client to port 8080,  both services and clients are given TCP protocols and are equipped with ClusterIPs. The mongo database is also equipped with a ClusterIP with TCP protocol but it publishes to port 27017. The deployment.yaml file creates a container for each component using the corresponding image obtained from DockerHub. As well the environment variables are once again set for the email and authentication services. The service and deployment for the gateway equip it with a LoadBalancer and a published port of 80. The set-up is then run in the folder containing the services and deployments:
```
kubectl apply -f ./
```
Success of the procedure should be demonstrated by a corresponding working website

To implement Continuous Integration, I use Jenkins. I create a Jenkins folder with a dockerfile and 5 yaml files and within the folder run the 'kubectl apply ...' command. I then obtain the jenkins pod and log into jenkins from there. This allows me to run a shell script within jenkins that can act on the images within my DockerHub page
