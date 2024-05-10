# Python

[Source code :simple-github:](https://github.com/OpenVidu/openvidu-livekit-tutorials/tree/master/application-server/python){ .md-button target=\_blank }

This is a minimal server application built for Python with [Flask](https://flask.palletsprojects.com/){:target="\_blank"} that allows generating LiveKit tokens on demand.

It internally uses [LiveKit Python SDK](https://github.com/livekit/python-sdks){:target="\_blank"}.

## Running this application

#### Prerequisites

To run this application you will need **Python 3**:

- [Python 3](https://www.python.org/downloads/){:target="\_blank"}

#### Download repository

```bash
git clone https://github.com/OpenVidu/openvidu-livekit-tutorials.git
cd openvidu-livekit-tutorials/application-server/python
```

#### Create a python environment

You can create a new pyhton environment with the aim of isolating the dependencies of this application:

```bash
python3 -m venv venv
. venv/bin/activate
```

#### Install dependencies

```bash
pip install -r requirements.txt
```

#### Run application

```bash
python3 app.py
```

## Understanding the code

The application is a simple Flask app with a single controller file `app.py` that exports a unique endpoint:

- `/token` : Generate a token for a given Room name and Participant name

Let's see the code of the `app.py` file:

```python title="<a href='https://github.com/OpenVidu/openvidu-livekit-tutorials/blob/master/application-server/python/app.py#L1-L15' target='_blank'>app.py</a>" linenums="1"
import os
from flask import Flask, request, jsonify
from flask_cors import CORS
from dotenv import load_dotenv
from livekit import api # (1)!

load_dotenv() # (2)!

SERVER_PORT = os.environ.get("SERVER_PORT", 6080) # (3)!
LIVEKIT_API_KEY = os.environ.get("LIVEKIT_API_KEY", "devkey") # (4)!
LIVEKIT_API_SECRET = os.environ.get("LIVEKIT_API_SECRET", "secret") # (5)!

app = Flask(__name__) # (6)!

CORS(app) # (7)!
```

1. Import `api` from `livekit` library
2. Load environment variables from `.env` file
3. The port where the application will be listening
4. The API key of LiveKit Server
5. The API secret of LiveKit Server
6. Initialize the Flask application
7. Enable CORS support

The `app.py` file imports the required dependencies and loads the necessary environment variables from `.env` file using `dotenv` library:

- `SERVER_PORT`: the port where the application will be listening.
- `LIVEKIT_API_KEY`: the API key of LiveKit Server.
- `LIVEKIT_API_SECRET`: the API secret of LiveKit Server.

Finally the `Flask` application is initialized and CORS support is enabled.

#### Create token endpoint

The unique endpoint of the application is `/token`. It receives a JSON object with the following fields:

- `roomName`: the name of the Room where the user wants to connect.
- `participantName`: the name of the participant that wants to connect to the Room.

```python title="<a href='https://github.com/OpenVidu/openvidu-livekit-tutorials/blob/master/application-server/python/app.py#L18-L31' target='_blank'>app.py</a>" linenums="18"
@app.post("/token")
def getToken():
    room_name = request.json.get("roomName")
    participant_name = request.json.get("participantName")

    if not room_name or not participant_name:
        return jsonify("roomName and participantName are required"), 400

    token = (
        api.AccessToken(LIVEKIT_API_KEY, LIVEKIT_API_SECRET) # (1)!
        .with_identity(participant_name) # (2)!
        .with_grants(api.VideoGrants(room_join=True, room=room_name)) # (3)!
    )
    return jsonify(token.to_jwt()) # (4)!
```

1. A new `AccessToken` is created providing the `LIVEKIT_API_KEY` and `LIVEKIT_API_SECRET`.
2. We set participant's identity in the AccessToken.
3. We set the video grants in the AccessToken. `room_join` allows the user to join a room and `room` determines the specific room. Check out all [Video Grants](https://docs.livekit.io/realtime/concepts/authentication/#Video-grant){:target="\_blank"}.
4. Finally, we convert the AccessToken to a JWT token and send it back to the client.

The endpoint first obtains the `roomName` and `participantName` parameters from the request body. If they are not available, it returns a `400` error.

If required fields are available, a new JWT token is created. For that we use the [LiveKit Python SDK](https://github.com/livekit/python-sdks){:target="\_blank"}:

1. A new `AccessToken` is created providing the `LIVEKIT_API_KEY` and `LIVEKIT_API_SECRET`.
2. We set participant's identity in the AccessToken.
3. We set the video grants in the AccessToken. `room_join` allows the user to join a room and `room` determines the specific room. Check out all [Video Grants](https://docs.livekit.io/realtime/concepts/authentication/#Video-grant){:target="\_blank"}.
4. Finally, we convert the AccessToken to a JWT token and send it back to the client.
