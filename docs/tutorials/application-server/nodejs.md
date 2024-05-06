

# openvidu-basic-node

[Source code :simple-github:](https://github.com/OpenVidu/openvidu-livekit-tutorials){ .md-button target=_blank }

This is a minimal server application sample built for Node with [Express](https://expressjs.com/){:target="_blank"}.
It internally uses [livekit-server-sdk](https://docs.livekit.io/server-sdk-js/index.html){:target="_blank"}.

## Running this application

#### Prerequisites
To run this application you will need **Node**:

- [Node](https://nodejs.org/es/download/){:target="_blank"}

#### Download repository

```bash
git clone https://github.com/OpenVidu/openvidu-livekit-tutorials.git
cd openvidu-livekit-tutorials/openvidu-basic-node
```

#### Install dependencies

```bash
npm install
```

#### Run application

```bash
node index.js
```

## Understanding the code

The application is a simple Express app with a single controller file `index.js` that exports an unique endpoint:

- `/token` : Generate a token for a given Room name and participant name

<!-- > You can get more information about these endpoints in the [Application Server Endpoints](application-server/#rest-endpoints) section. -->

Let's see the code of the controller:

```javascript
var AccessToken = require('livekit-server-sdk').AccessToken;
var cors = require("cors");
var app = express();

// Environment variable: PORT where the node server is listening
var SERVER_PORT = process.env.SERVER_PORT || 6080;
// Environment variable: api key shared with our LiveKit deployment
var LIVEKIT_API_KEY = process.env.LIVEKIT_API_KEY || 'devkey';
// Environment variable: api secret shared with our LiveKit deployment
var LIVEKIT_API_SECRET = process.env.LIVEKIT_API_SECRET || 'secret';


// Enable CORS support
app.use(
  cors({
    origin: "*",
  })
);

var server = http.createServer(app);

// Serve application
server.listen(SERVER_PORT, () => {
	console.log('Application started on port: ', SERVER_PORT);
});
```

Starting by the top, the `index.js` file has the following fields:

- `cors`: allows the application to be accessed from any domain.
- `app`: the Express application.
- `server`: the HTTP server.
- `AccessToken`: the `AccessToken` object that will be used to creates a new Accesstoken for being able to connect to the LiveKit server. It is initialized with the `LIVEKIT_API_KEY` and `LIVEKIT_API_SECRET` environment variables.
- `SERVER_PORT`: the port where the application will be listening.
- `LIVEKIT_API_KEY`: the api key shared with our LiveKit deployment.
- `LIVEKIT_API_SECRET`: the api secret shared with our LiveKit deployment.

<br>

#### Create token endpoint

The unique endpoint of the application is `/token`. It receives a JSON object with the following fields:

- `roomName`: the name of the room where the user wants to connect.
- `participantName`: the name of the participant that wants to connect to the room.

```javascript
app.post('/token', (req, res) => {
	const roomName = req.body.roomName;
	const participantName = req.body.participantName;

	if (!roomName || !participantName) {
		res.status(400).send('roomName and participantName are required');
		return;
	}

	const at = new AccessToken(LIVEKIT_API_KEY, LIVEKIT_API_SECRET, {
		identity: participantName,
	});
	at.addGrant({ roomJoin: true, room: roomName });
	const token = at.toJwt();
	res.send(token);
});
```

The endpoint first checks if the request body has the `roomName` and `participantName` fields. If not, it returns a `400` error.

Then, it creates a new `AccessToken` object with the `LIVEKIT_API_KEY` and `LIVEKIT_API_SECRET` environment variables. This object will be used to create a new token for the user.

After that, it adds a new grant to the `AccessToken` object. This grant will allow the user to connect to the room specified in the request body.

Finally, the `AccessToken` is converted to a JWT token and returned in the response body.
