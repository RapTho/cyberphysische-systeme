# Subscriber App

## Introduction

This is a Python subscriber app that connects to a mosquitto broker and subscribes to a specific topic. The app receives messages from the broker and stores them in a Cloudant database.

## Prerequisites

- mosquitto broker deployed on IBM Code Engine (previous task)
- Python 3.8 or later
- `paho-mqtt` library for MQTT communication
- `requests` library for HTTP requests
- `python-dotenv` library for loading environment variables from a .env file

## Task

Your task is to develop a Python subscriber app that:

- Subscribes to a topic on the previously deployed mosquitto broker on port 8083 (WebSocket)
- Receives messages from the broker and stores them in a Cloudant database
- Enforce a message schema

## Hints

- Use the paho-mqtt library to connect to the mosquitto broker and subscribe to a topic.
- Use the requests library to send HTTP requests to the Cloudant database API.
- Use basic authentication to authenticate with the Cloudant database.
- Load environment variables from a .env file using the python-dotenv library.
- Use a try-except block to handle errors when connecting to the mosquitto broker or publishing messages.

## Cloudant Database API

The Cloudant database API provides a RESTful interface for interacting with the database. You can use the `requests` library to send HTTP requests to the API.

To create a new document, send a POST request to https://<cloudant-account>.cloudant.com/<database-name>.
To retrieve a document, send a GET request to https://<cloudant-account>.cloudant.com/<database-name>/<document-id>.

### Authentication

To authenticate with the Cloudant database, you need to exchange an API Key to a Bearer token. Check out [this chapter](7_Cloudant.md#token-exchange) for more information.

### Example Code

Here is an example of how you can use the paho-mqtt library to connect to the mosquitto broker and subscribe to a topic. More documentation is available [here](https://eclipse.dev/paho/files/paho.mqtt.python/html/client.html)

```python
import ssl
import paho.mqtt.client as mqtt

topic = "topic1"
client = mqtt.Client(transport="websockets") # or mqtt.CallbackAPIVersion.VERSION2
client.tls_set(None, cert_reqs=ssl.CERT_NONE) # if you want to ignore TLS
client.username_pw_set("myBrokerUsername", "myBrokerPassword")
client.connect("mosquitto-broker-url", 443)
client.subscribe(topic, qos=1) # qos=1 is at least once
```

Validating a message

```python
import jsonschema

schema = {
    "type": "object",
    "properties": {
        "id": {"type": "integer"},
        "message": {"type": "string"}
    },
    "required": ["id", "message"]
}

message = {"id": 1, "message": "Hello, world!"}
jsonschema.validate(message, schema)
```

Write logs to standard output (stdout) as documented [here](https://eclipse.dev/paho/files/paho.mqtt.python/html/client.html#paho.mqtt.client.Client.on_log)

```python
from datetime import datetime

def on_log(client, userdata, level, buf):
    print(f"[{datetime.now()}] MQTT Log: {buf}")

client.on_log = on_log
```

Here is an example of how you can use the requests library to send a POST request to the Cloudant database API:

```python
import requests

url = "https://<cloudant-host>/<database-name>"
auth = ("myCloudantUsername", "myCloudantPassword")
headers = {"Content-Type": "application/json"}
data = {"key": "value"}
response = requests.post(url, auth=auth, headers=headers, json=data)
```

Load environment variables from a .env file using the python-dotenv library. For example:

```python
import os
from dotenv import load_dotenv

load_dotenv()
BROKER_URL = os.getenv("BROKER_URL")
CLOUDANT_HOST = os.getenv("CLOUDANT_HOST")
CLOUDANT_DATABASE = os.getenv("CLOUDANT_DATABASE")
```

## Deployment on IBM Code Engine

The container image building and publishing process will be similar to what was documented in the [mosquitto example](./3_Deploy_on_IBM-Cloud.md). The deployment of the container on IBM Code Engine will be slightly different though.

Instead of an `app` we'll deploy a `job` in the `daemon` mode. This job will run forever (until stopped) and not expose any port.

First, you need to create your subscriber specific `configMap` and `secret`

**Unix/macOS:**

```bash
ibmcloud ce configmap create --name subscriber-conf-${USER} --from-literal key1=value1 --from-literal key2=value2
ibmcloud ce secret create --name subscriber-secret-${USER} --from-literal key1=value1 --from-literal key2=value2
```

**Windows (PowerShell):**

```powershell
ibmcloud ce configmap create --name subscriber-conf-$env:USERNAME --from-literal key1=value1 --from-literal key2=value2
ibmcloud ce secret create --name subscriber-secret-$env:USERNAME --from-literal key1=value1 --from-literal key2=value2
```

Then create the `job` in `daemon` mode

**Unix/macOS:**

```bash
ibmcloud ce job create --mode daemon --name subscriber-${USER} --image de.icr.io/${CR_NAMESPACE}/${IMAGE_NAME}:${IMAGE_TAG} --registry-secret ibm-container-registry-${USER} --env-from-configmap subscriber-conf-${USER} --env-from-secret subscriber-secret-${USER} --cpu 0.25 --memory 0.5G
```

**Windows (PowerShell):**

```powershell
ibmcloud ce job create --mode daemon --name subscriber-$env:USERNAME --image de.icr.io/${env:CR_NAMESPACE}/${env:IMAGE_NAME}:${env:IMAGE_TAG} --registry-secret ibm-container-registry-$env:USERNAME --env-from-configmap subscriber-conf-$env:USERNAME --env-from-secret subscriber-secret-$env:USERNAME --cpu 0.25 --memory 0.5G
```

Finally, start the job

**Unix/macOS:**

```bash
ibmcloud ce jobrun submit --name subscriber-run-${USER} --job subscriber-${USER}
```

**Windows (PowerShell):**

```powershell
ibmcloud ce jobrun submit --name subscriber-run-$env:USERNAME --job subscriber-$env:USERNAME
```

> **Note:** For more details on Windows environment variables and command patterns, see the [Windows Environment Setup Guide](./8_Windows-Environment-Setup.md).

More documentation can be found [here](https://cloud.ibm.com/docs/codeengine?topic=codeengine-job-daemon)
