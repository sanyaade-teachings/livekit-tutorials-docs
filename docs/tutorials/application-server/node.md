# Node

[Source code :simple-github:](https://github.com/OpenVidu/openvidu-livekit-tutorials/tree/master/application-server/node){ .md-button target=\_blank }

This is a minimal server application built for Node with [Express](https://expressjs.com/){:target="\_blank"} that allows generating LiveKit tokens on demand.

It internally uses [LiveKit JS SDK](https://docs.livekit.io/server-sdk-js){:target="\_blank"}.

## Running this application

#### Prerequisites

To run this application you will need **Node**:

- [Node](https://nodejs.org/es/download/){:target="\_blank"}

#### Download repository

```bash
git clone https://github.com/OpenVidu/openvidu-livekit-tutorials.git
cd openvidu-livekit-tutorials/application-server/node
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

The application is a simple Express app with a single file `index.js` that exports a unique endpoint:

- `/token` : Generate a token for a given Room name and Participant name

Let's see the code of the `index.js` file:

```javascript title="<a href='https://github.com/OpenVidu/openvidu-livekit-tutorials/blob/master/application-server/node/index.js#L1-L13' target='_blank'>index.js</a>" linenums="1"
import "dotenv/config";
import express from "express";
import cors from "cors";
import { AccessToken } from "livekit-server-sdk"; // (1)!

const SERVER_PORT = process.env.SERVER_PORT || 6080; // (2)!
const LIVEKIT_API_KEY = process.env.LIVEKIT_API_KEY || "devkey"; // (3)!
const LIVEKIT_API_SECRET = process.env.LIVEKIT_API_SECRET || "secret"; // (4)!

const app = express(); // (5)!

app.use(cors()); // (6)!
app.use(express.json()); // (7)!
```

1. Import `AccessToken` from `livekit-server-sdk`
2. The port where the application will be listening
3. The API key of LiveKit Server
4. The API secret of LiveKit Server
5. Initialize the Express application
6. Enable CORS support
7. Enable JSON body parsing

The `index.js` file imports the required dependencies and loads the necessary environment variables:

- `SERVER_PORT`: the port where the application will be listening.
- `LIVEKIT_API_KEY`: the API key of LiveKit Server.
- `LIVEKIT_API_SECRET`: the API secret of LiveKit Server.

Finally the `express` application is initialized and CORS and JSON body parsing are enabled.

#### Create token endpoint

The unique endpoint of the application is `/token`. It receives a JSON object with the following fields:

- `roomName`: the name of the Room where the user wants to connect.
- `participantName`: the name of the participant that wants to connect to the Room.

```go title="<a href='https://github.com/OpenVidu/openvidu-livekit-tutorials/blob/master/application-server/node/index.js#L15-L30' target='_blank'>index.js</a>" linenums="15"
app.post("/token", async (req, res) => {
  const roomName = req.body.roomName;
  const participantName = req.body.participantName;

  if (!roomName || !participantName) {
    res.status(400).json("roomName and participantName are required");
    return;
  }

  const at = new AccessToken(LIVEKIT_API_KEY, LIVEKIT_API_SECRET, { // (1)!
    identity: participantName,
  });
  at.addGrant({ roomJoin: true, room: roomName }); // (2)!
  const token = await at.toJwt(); // (3)!
  res.json(token); // (4)!
});
```

1. A new `AccessToken` is created providing the `LIVEKIT_API_KEY`, `LIVEKIT_API_SECRET` and setting the participant's identity.
2. We set the video grants in the AccessToken. `roomJoin` allows the user to join a room and `room` determines the specific room. Check out all [Video Grants](https://docs.livekit.io/realtime/concepts/authentication/#Video-grant){:target="\_blank"}.
3. We convert the AccessToken to a JWT token.
4. Finally, the token is sent back to the client.

The endpoint first obtains the `roomName` and `participantName` parameters from the request body. If they are not available, it returns a `400` error.

If required fields are available, a new JWT token is created. For that we use the [LiveKit JS SDK](https://docs.livekit.io/server-sdk-js){:target="\_blank"}:

1. A new `AccessToken` is created providing the `LIVEKIT_API_KEY`, `LIVEKIT_API_SECRET` and setting the participant's identity.
2. We set the video grants in the AccessToken. `roomJoin` allows the user to join a room and `room` determines the specific room. Check out all [Video Grants](https://docs.livekit.io/realtime/concepts/authentication/#Video-grant){:target="\_blank"}.
3. We convert the AccessToken to a JWT token.
4. Finally, the token is sent back to the client.
