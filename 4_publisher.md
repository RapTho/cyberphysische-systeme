# Publisher App

## Introduction

This is a Python publisher app that connects to a mosquitto broker and publishes messages to a specific topic. The app uses the paho-mqtt library to communicate with the mosquitto broker.

## Prerequisites

- mosquitto broker deployed on IBM Code Engine (previous task)
- Python 3.8 or later
- `paho-mqtt` library for MQTT communication
- `jsonschema` library for validating JSON messages
- `python-dotenv` library for loading environment variables from a .env file

## Task

Your task is to develop a Python publisher app that:

- Connects to the mosquitto broker on port 8083 (WebSocket)
- Validates the JSON messages using a schema
- Publishes messages to a specific topic (e.g., student1/topic)

## Hints

- Use the paho-mqtt library to connect to the mosquitto broker and publish messages to a topic.
- Use the jsonschema library to validate the JSON messages using a schema.
- Use the python-dotenv library to load environment variables from a .env file.
- Use a try-except block to handle errors when connecting to the mosquitto broker or publishing messages.

## Schema validation

Schema validation ensures that the data structure and format of messages are consistent across all publishers and subscribers in the system. This validation is performed at the **application level** (in your Python code), not by the MQTT protocol or the broker itself. By implementing schema validation, publishers and subscribers can comply with a mutually agreed message format, preventing data inconsistencies and communication errors between different components of the system.

When multiple applications exchange messages through MQTT, they need to agree on a common message structure. Schema validation helps enforce this agreement by checking that each message contains the required fields with the correct data types before it is published or after it is received. This is particularly important when working with JSON payloads, as JSON is flexible and doesn't enforce structure on its own.

You can use the `jsonschema` library to validate the JSON messages:

- Define a schema using the jsonschema library
- Use the validate function to validate the JSON messages against the schema

## Example Code

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

publish a message

```python
from datetime import datetime

topic = "topic1"
result = client.publish(topic, payload=json.dumps(message), qos=1)
result_code = result.rc
if result_code != mqtt.MQTT_ERR_SUCCESS:
    raise Exception(f"[{datetime.now()}] Failed to publish message '{message}' to topic ${topic}. Result code: {result_code}")
else:
    print(f"[{datetime.now()}] Message '{message}' sent to topic ${topic} successfully.")
```

## Environment variables

Environment variables are key-value pairs stored outside your code that configure application behavior. They allow you to change settings (like URLs, credentials, or feature flags) without modifying the code itself.

**Benefits:**

- **Security**: Keep sensitive data (passwords, API keys) out of your codebase
- **Flexibility**: Change configuration across environments (development, production) without code changes
- **Portability**: Same code works in different environments with different configurations

Load environment variables from a .env file using the python-dotenv library. For example:

```python
import os
from dotenv import load_dotenv

load_dotenv()
BROKER_URL = os.getenv("BROKER_URL")
USERNAME = os.getenv("USERNAME")
PASSWORD = os.getenv("PASSWORD")
```

**Note:** When running in a container, environment variables must be explicitly passed to the container, for example using [podman run --env-file .env](https://docs.podman.io/en/latest/markdown/podman-run.1.html#env-file-file) or by setting individual variables with `--env KEY=value`.
