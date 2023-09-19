# Overview 
The SRT FBO Scraper is a Python application that gathers data about Information Technology (IT) solicitations submitted by agencies around the federal government by scraping that data from the SAM.gov website. For each solicitation that is found, this application extracts the text of each document and feeds it to a supervised machine learning model in order to determine whether or not the document contains appropriate Section 508 accessibility language. 

Following a service-oriented architecture, this application comprises one component of the back-end for the Solicitation Review Tool, a web application that GSA policy experts will use to review IT solicitations for Section 508 compliance; notify the owners of deficient solicitations; monitor historical changes in compliance; and validate predictions to improve the machine-learning model's performance. 

This application is designed to run as a cron daemon within a Docker image on cloud.gov. Here's what happens every time the cron job is triggered: 
1) Fetches yesterday's updated/posted solicitations from sam.gov using the Opportunity Management API, filtering for those solicitations that have an IT-related NAICS code.
2) Uses the Federal Hierarchy API to look up canonical agency and office names. 
3) For each solicitation, it downloads a zip archive containing all of the solicitation's relevant documents using the Opportunity Management API. 
4) Extracts the text from each of those documents using textract. 
5) Restructures the data and inserts it into a PostgreSQL database. 

In a future release, the script will poll the database for the number of user-validated predictions on document compliance. If there's a significant increase, those newly validated documents will be sent to the machine learning component of the application to train a new and improved model. 
# Developer Requirements 
## Software Components and Tools 
The following is a summary of the software and tools that are needed for development of this project: 
* Operating system - Linux, Ubuntu, Mac OS, Windows 
* IDE - Visual Studio Code, etc. 
* Python 3.6 
* Docker 
* PostGreSQL 
* SNYK 
* GitHub 
* Node 
* Node Package Manager 
* Yarn 
## Systems Access 
Access to the following platforms will also be required for development: 
* Cloud.gov 
* SAM.gov 
* MAX.gov 
* Docker 
* SNYK 
* GitHub - GSA team 
## Environment Variables 
* SAM_API_KEY 
* SAM_API_URI 
* VCAP_SERVICES
* VCAP_APPLICATION 
* TEST_DB_URL 
* SUPERCRONIC 
# Setup and Deployment 
This application is designed to work as a dockerized daemon in cloud.gov. As a cloud.gov app, it is bound with a postgres database that is provided as a brokered service. See the Cloud.gov Deployment section below if you are interested in pushing the application to cloud.gov. 
## Create a sam.gov account
No matter how you plan on running this application, you will need to create both a personal (i.e. Public) and System Account in either sam.gov or alpha.sam.gov (the latter is their dev version, so you should opt for an account with the former). Instructions for getting those accounts set up can be found here. 

Note: the system account will require you to specify the ip address(es) from which you plan to access the APIs. If you only plan on accessing the APIs from within cloud.gov, you can list the external IP address ranges that cloud.gov uses. For local access, you will need to add your own external ip address. 
## Set Environment Variables 
Assuming you are using sam.gov's APIs, after you have set up your personal account, generate a public API key (this is for the Federal Hierarchy API) and set it as an environment variable (locally and/or in cloud.gov) as SAM_API_KEY. 
## Installation
* To get started with SRT-API, go to [GSA/srt-api](https://github.com/GSA/srt-fbo-scraper) to copy the URL for cloning the project. 
* Open Terminal or use Visual Studio Code and open a terminal window. 
* Navigate to the desired folder and clone the project. 
* Next navigate to the bin folder that was created through the clone. 
* Type `./dev_setup.sh` to begin setup 
* This script will install and set up much of what you need for this project: 
    * Node Package Manager (npm) 
    * PostgreSQL 
    * Node Version Manager (nvm) 
    * SNYK - log into SNYK using your GitHub account when prompted by the script 
* This script will also create the needed local Postgres database with all of the tables required by this project, if they do not already exist. 
* It will also install Node Version 16, which is required for this project. 
* Finally it will install and update all of the Node modules used in this project. 
## Cloud.gov Deployment
Build the Docker Image
Before pushing to cloud.gov, you need to build the Docker image and push it to DockerHub. 

To build and push the image: 
```
DOCKER_USER=<your user name>
DOCKER_PASS=<your password>
TARGET_SPACE=<dev, staging or prod> #choose one
docker build -t $DOCKER_USER/srt-fbo-scraper-$TARGET_SPACE . 
echo "$DOCKER_PASS" | docker login --username $DOCKER_USER --password-stdin    
docker push $DOCKER_USER/srt-fbo-scraper-$TARGET_SPACE 
```
Push to cloud.gov
Log into cloud.gov with: 

    cf login -a api.fr.cloud.gov --sso
Target the appropriate org and space (i.e.TARGET_SPACE from above) by following the prompts. 

If this is your first time pushing the application, you need to first create and bind a postgres service instance to the application: 
```
cf create-service <service> <service-tag>
#wait a few minutes for the service to be provisioned
cf create-service-key <service-tag> <service-key-name>    #if this returns an OK, then your service has been provisioned  
cf push srt-fbo-scraper --docker-image $DOCKER_USER/srt-fbo-scraper-$TARGET_SPACE
cf bind-service srt-fbo-scraper <service-tag>  
cf restage srt-fbo-scraper
```
Above, `<service>` is the name of a postgres service (e.g. `aws-rds shared-psql`) while `<service-tag>` is whatever you want to call this service. 

Since services can sometimes take up to 60 minutes to be provisioned, we use cf create-service-key to ensure the service has been provisioned. See this for more details. 

Every subsequent time you can merely use: 
```
cf push srt-fbo-scraper --docker-image $DOCKER_USER/srt-fbo-scraper-$TARGET_SPACE 
```
# More Info  
For more detailed information, please go here: [Documentation](https://github.com/GSA/srt-fbo-scraper/tree/main/documentation)
