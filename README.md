# Operation

This repository contains the deployment configuration, including a `docker-compose.yml`, and links to all other components.

## About
This repository contains all information about running the project. It is an adaptation of the [Restaurant Sentiment Analysis](https://github.com/proksch/restaurant-sentiment) project. 

## Relevant repositories
Repository | Purpose | 
| --- | --- | 
[model-training](https://github.com/remla25-team12/model-training) | Training pipeline for the restaurant sentiment analysis model. |
[model-service](https://github.com/remla25-team12/model-service) | Model wrapper for communication with other components of the project, e.g. for inference | 
[lib-ml](https://github.com/remla25-team12/model-service) | Preprocessing logic for input data, used during model training and inference. | 
[lib-version](https://github.com/remla25-team12/lib-version) | A simple version-aware libary that reports its own version. | 
[app](https://github.com/remla25-team12/app) | Webapp (frontend + service) to interface with the model | 

# Getting started
## Requirements
- Linux or MacOS
- Docker


## Install and run
1. Clone this repository, and all other repositories listed in "Relevant repositories", into a folder.
2. Deploy the project using Docker Compose by running this command in the project's root folder.
```
$ docker-compose up -d
```
3. Navigate to localhost:5000 to access the application homepage.

## Config
The .env file in this repository allows you to choose which version of the app and model-service container you'd like to use. By default, the latest image will be pulled.


# Continuous progress log
## Assignment 1
[Add a new paragraph for each assignment as a continuous progress log that (briefly) describes which assignment parts have been implemented to support the peer-review process.]



