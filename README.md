# Operation
REMLA Group 12

## About
This project is an adaptation of the [Restaurant Sentiment Analysis](https://github.com/proksch/restaurant-sentiment) project. 
This repository serves as the central point of the project, containing the deployment configuration (`docker-compose.yaml`), installation instructions, and links to other components.

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
- Docker and Docker Compose

## Install and run
1. Clone this repository.
2. Deploy the project using Docker Compose by running this command in the project's root folder.
```
$ docker compose up -d
```
3. Navigate to `http://localhost:8080` to access the application homepage.

## Config
The .env file in this repository allows you to choose which version of the `app` and `model-service` container you'd like to use. If none are specified, the latest images will be pulled by default.


# Continuous progress log
_Add a new paragraph for each assignment as a continuous progress log that (briefly) describes which assignment parts have been implemented to support the peer-review process._

## Assignment 1
The docker-compose.yaml successfully launches both containers (`app` and `model-service`). They communicate with each other over a shared Docker network, and only `app`'s port is exposed to the host. The app frontend can be used to query the model for predictions. All libraries and containers are released using automatic versioning via a GitHub workflow. We do not yet have additional interaction options (e.g. flagging incorrect predictions).





