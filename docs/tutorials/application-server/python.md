# openvidu-basic-python

[Source code :simple-github:](https://github.com/OpenVidu/openvidu-livekit-tutorials){ .md-button target=_blank }

This is a minimal server application sample built for Python with [Flask](https://flask.palletsprojects.com/){:target="\_blank"}.
It internally uses the [livekit-server-sdk-python](https://github.com/livekit/livekit-server-sdk-python).

## Running this application

#### Prerequisites

To run this application you will need **Python 3**:

- [Python 3](https://www.python.org/downloads/){:target="\_blank"}

#### Download repository

```bash
git clone https://github.com/OpenVidu/openvidu-livekit-tutorials.git
cd openvidu-livekit-tutorials/openvidu-basic-python
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

The application is a simple Flask application with a single controller file `app.py` that exports a unique endpoint:

- `/token` : Generate a token for a given Room name and Participant name

Let's see the code of the controller:

```python
app = Flask(__name__)

# Enable CORS support
cors = CORS(app, resources={r"/*": {"origins": "*"}})

# Load env variables
SERVER_PORT = os.environ.get("SERVER_PORT")
LIVEKIT_API_KEY = os.environ.get("LIVEKIT_API_KEY")
LIVEKIT_API_SECRET = os.environ.get("LIVEKIT_API_SECRET")

if __name__ == "__main__":
    app.run(debug=True, host="0.0.0.0", port=SERVER_PORT)
```

Starting by the top, the `app.py` file has the following fields:

- `app`: Flask application.
- `cors`: CORS support for Flask application.
- `SERVER_PORT`: port of the application server.
- `LIVEKIT_API_KEY`: the API key of the LiveKit deployment.
- `LIVEKIT_API_SECRET`: the API secret of the LiveKit deployment.

<br>

#### Create token endpoint

The unique endpoint of the application is `/token`. It receives a JSON object with the following fields:

- `roomName`: the name of the Room where the user wants to connect.
- `participantName`: the name of the participant that wants to connect to the Room.

```python
@app.route("/token", methods=['POST'])
def getToken():
    room_name = request.json.get("roomName")
    participant_name = request.json.get("participantName")

    if not room_name or not participant_name:
        return "roomName and participantName are required", 400

    # Create a VideoGrant with the necessary permissions
    grant = livekit.VideoGrant(room_join=True, room=room_name)

    # Create an AccessToken with your API key, secret and the VideoGrant
    access_token = livekit.AccessToken(LIVEKIT_API_KEY, LIVEKIT_API_SECRET, grant=grant, identity=participant_name)

    # Generate the token
    return access_token.to_jwt()
```

The endpoint first checks if the request body has the `roomName` and `participantName` fields. If not, it returns a `400` error.

Then, it creates a new `VideoGrant` object with the necessary permissions. In this case, the grant allows the user to join the room specified in the request body.

After that, it creates a new `AccessToken` object with the `LIVEKIT_API_KEY` and `LIVEKIT_API_SECRET` environment variables, the `VideoGrant` object, and the participant name.

Finally, the `AccessToken` is converted to a JWT token and returned in the response body.
