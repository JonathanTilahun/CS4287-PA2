# CS4287-Assignment2: IoT Data Analytics Pipeline with Ansible and Docker

## Overview
This project builds upon Assignment 1, further implementing a cloud-based IoT data analytics pipeline using Infrastructure-as-Code (IaC) with Ansible and containerizing components using Docker. The scenario emulates an IoT application where surveillance cameras send data to Kafka brokers, which are then processed by machine learning (ML) models and stored in a MongoDB database. The system consists of producers, consumers, a Kafka broker, a MongoDB database, and an ML inference server.

## Technologies Used
- **Ansible**: For automating the provisioning and setup of VMs on Chameleon Cloud.
- **Apache Kafka**: For streaming and managing the message queue between the producer (IoT devices) and consumers.
- **MongoDB**: Used for storing the processed image data along with metadata.
- **Python**: Main programming language, including libraries like `kafka-python`, `pymongo`, and `Flask` for different components.
- **Flask**: Hosting the machine learning inference model (ResNet50).
- **TensorFlow**: Used for loading the pre-trained ResNet50 ML model.
- **Docker**: Containerized deployment for the Kafka Producer, Consumer, ML server, and MongoDB.
- **Chameleon Cloud**: Cloud platform where we deployed four virtual machines to distribute the different components of the pipeline.

## Project Architecture
The project is divided across four VMs, each hosting a containerized component of the system:

- **VM1**: IoT Producer (Dockerized)
- **VM2**: ML Model (ResNet50) Inference Server (Dockerized)
- **VM3**: Kafka Broker & Consumers (Dockerized)
- **VM4**: MongoDB Database (Dockerized)

### VM1: IoT Producer
- **Role**: Simulates IoT camera devices sending images to Kafka.
- **Details**:
  - Python script generates synthetic image data using CIFAR-10.
  - The data is serialized as JSON and sent to Kafka via the `kafka-python` library.
  - The message includes an `image_id`, `device_id`, `timestamp`, and the image data.

### VM2: ML Inference Server
- **Role**: Hosts the ResNet50 model using Flask and Gunicorn to perform real-time image inference.
- **Details**:
  - Flask API exposes an endpoint that accepts base64-encoded image data and returns a classification label.
  - The ML model is pre-trained on CIFAR-10 using TensorFlow's ResNet50 architecture.
  - Flask is deployed in a containerized environment using Gunicorn with multiple workers for handling concurrent requests.

### VM3: Kafka Broker & Consumers
- **Role**: Hosts the Kafka broker, responsible for managing the data pipeline between the IoT producer and the consumers.
- **Details**:
  - Kafka Broker version 2.13-3.8.0 is containerized and runs within a Docker container.
  - **Consumers**:
    - **DB Consumer**: Consumes the image data and stores it in MongoDB (hosted on VM4).
    - **Inference Consumer**: Sends the image data to the Flask API hosted on VM2 for inference, then updates MongoDB with the predicted label.

### VM4: MongoDB Database
- **Role**: Stores all incoming data from Kafka, including image metadata and inference results.
- **Details**:
  - MongoDB is containerized and runs within a Docker container.
  - The DB Consumer (on VM3) inserts the image metadata, and the Inference Consumer (on VM3) updates MongoDB with the inference label.

## Installation & Setup

### Prerequisites
- Ensure you have a Chameleon Cloud account and have created 4 VMs (VM1-VM4) using Ubuntu 22.04.
- Install Docker and Python 3 on each VM.

### Step-by-Step Setup

#### VM1: IoT Producer
1. Install Kafka Python Client inside the Docker container:
    ```bash
    pip install kafka-python
    ```

2. Build and run the Docker container for the IoT Producer:
    ```bash
    docker build -t producer .
    docker run producer
    ```

#### VM2: ML Inference Server
1. Install the necessary Python libraries inside the Docker container:
    ```bash
    pip install flask tensorflow gunicorn
    ```

2. Build and run the Docker container for the ML Inference Server:
    ```bash
    docker build -t ml-server .
    docker run ml-server
    ```

#### VM3: Kafka Broker & Consumers
1. Build and run the Docker container for Kafka Broker:
    ```bash
    docker build -t kafka-broker .
    docker run kafka-broker
    ```

2. Run the consumers:
    ```bash
    docker build -t consumer .
    docker run consumer
    ```

#### VM4: MongoDB
1. Build and run the Docker container for MongoDB:
    ```bash
    docker build -t mongo .
    docker run mongo
    ```

## How the System Works
1. **IoT Producer (VM1)** generates synthetic image data and sends it to the Kafka broker (VM3).
2. **Kafka Broker (VM3)** stores the messages and distributes them to two consumers: the DB Consumer and the Inference Consumer.
3. **DB Consumer (VM3)** inserts the image metadata into MongoDB (VM4).
4. **Inference Consumer (VM3)** sends the image to the ML Inference Server (VM2).
5. **ML Inference Server (VM2)** processes the image and sends back the inferred label to Kafka (VM3), which then updates MongoDB (VM4) with the predicted label.

## Project Structure
Assignment2/  
├── ansible_playbooks/    # Ansible playbooks for VM provisioning and package setup  
├── VM1/                  # IoT Producer Dockerfile and code  
├── VM2/                  # ML Inference Server Dockerfile and code  
├── VM3/                  # Kafka Broker and Consumers Dockerfile and code  
├── VM4/                  # MongoDB Dockerfile and code  
└── README.md             # This README file  


## Team Contributions
- **Jonathan Tilahun**: Dockerized the Kafka Producer and Consumer, worked on Ansible playbook integration, and managed the Kafka setup.
- **Dana Izadpanah**: Dockerized the ML Inference Server, developed Flask API for image classification, and tested communication with Kafka and MongoDB.
- **Micah Bronfman**: Developed the Ansible playbooks for automating VM provisioning and installing essential services (Kafka, Docker, MongoDB).

## Conclusion
This project successfully demonstrates the use of Ansible for VM provisioning and Docker for containerizing a cloud-based IoT data analytics pipeline. All components (Producer, Consumer, Kafka, and MongoDB) were deployed in Docker containers, ensuring scalability and portability.
