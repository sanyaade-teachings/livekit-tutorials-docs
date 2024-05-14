# openvidu-react

[Source code :simple-github:](https://github.com/OpenVidu/openvidu-livekit-tutorials){ .md-button target=\_blank }

This tutorial is a simple video-call application built with **React** which intents and purposes the same as [Livekit JavaScript tutorial](./javascript.md) but using **React** instead of plain web technologies.

## Running this tutorial

Running this tutorial is straightforward, and here's what you'll need:

### 1. Run LiveKit Server

--8<-- "docs/tutorials/shared/run-livekit-server.md"

### 2. Get the tutorial code

```bash
git clone https://github.com/OpenVidu/openvidu-livekit-tutorials.git
```

### 3. Run a server application

--8<-- "docs/tutorials/shared/application-server-tabs.md"

### 4. Run the client application

To run the client application tutorial, you'll need [NPM](https://docs.npmjs.com/downloading-and-installing-node-js-and-npm){:target="\_blank"} installed on your development computer.

Check if you have installed it by running the following command:

```bash
npm -v
```

Once you've confirmed that NPM is installed, you can proceed with the tutorial by following these steps:

1. Navigate into the application client directory:

    ```bash
    cd openvidu-livekit-tutorials/application-client/openvidu-react
    ```

2. Install dependencies:

    ```bash
    npm install
    ```

3. Run the application:

    ```bash
    npm run dev
    ```

After the server is up and running, you can test the application by visiting [`http://localhost:5080`](http://localhost:5080){:target="\_blank"}. You should see a screen like this:

<div class="grid-container">

<div class="grid-50"><p><a class="glightbox" href="../../../../assets/images/insecure-join.png" data-type="image" data-width="100%" data-height="auto" data-desc-position="bottom"><img src="../../../../assets/images/insecure-join.png" loading="lazy"/></a></p></div>

<div class="grid-50"><p><a class="glightbox" href="../../../../assets/images/insecure-session.png" data-type="image" data-width="100%" data-height="auto" data-desc-position="bottom"><img src="../../../../assets/images/insecure-session.png" loading="lazy"/></a></p></div>

</div>

--8<-- "docs/tutorials/shared/testing-other-devices.md"

## Understanding the code

This React project has been created using Vite and TypeScript. You can find more information about Vite [here](https://vitejs.dev/){:target="\_blank"}. You may come across various configuration files and other items that aren't essential for our tutorial. Our focus will be on the key files located within the `src/` directory

- `App.tsx`: This file defines the _App_ component, which serves as the main component of the application. It's responsible for handling tasks such as joining a video call and managing the video calls themselves.
- `App.css`: The CSS file linked to _App_, which controls the styling and appearance of the application's main component.
- `OvVideo.tsx`: This file introduces _OvVideo_. This component is responsible for encapsulating the `<video>` element that ultimately displays the video track within the application.
- `OvAudio.tsx`: This file introduces _OvAudio_. This component is responsible for encapsulating the `<audio>` element that ultimately displays the audio track within the application.

---

<h3 markdown>Using `livekit-client`</h3>

To understand how the `livekit-client` NPM package is used in `App.tsx`, let's break it down step by step. The code in this section provides an overview of how this package is utilized.

In the beginning, we import several essential objects from the `livekit-client` package that will be used throughout the code. These objects include:

```javascript
import {
  LocalTrackPublication,
  RemoteTrack,
  RemoteTrackPublication,
  Room,
  RoomEvent,
} from "livekit-client";
```

These imported objects are crucial for establishing and managing video calls. They represent different components and events associated with the video call system. We'll use them in subsequent parts of our code to create and manage video calls.

<h3 markdown>Defining variables</h3>

After importing the required objects, we define a few variables that will be used later in the code:

```typescript
const App = () => {
  // For local development, leave these variables empty
  // For production, configure them with correct URLs depending on your deployment
  let APPLICATION_SERVER_URL = "";
  let LIVEKIT_URL = "";

  // If APPLICATION_SERVER_URL is not configured, use default value from local development
  if (!APPLICATION_SERVER_URL) {
    if (window.location.hostname === "localhost") {
      APPLICATION_SERVER_URL = "http://localhost:6080/";
    } else {
      APPLICATION_SERVER_URL = "https://" + window.location.hostname + ":6443/";
    }
  }

  // If LIVEKIT_URL is not configured, use default value from local development
  if (!LIVEKIT_URL) {
    if (window.location.hostname === "localhost") {
      LIVEKIT_URL = "ws://localhost:7880/";
    } else {
      LIVEKIT_URL = "wss://" + window.location.hostname + ":7443/";
    }
  }

  const [room, setRoom] = useState<Room | undefined>(undefined);
  const [myMainPublication, setMyMainPublication] = useState<
    LocalTrackPublication | RemoteTrackPublication | undefined
  >(undefined);
  const [localPublication, setLocalPublication] = useState<
    LocalTrackPublication | undefined
  >(undefined);
  const [remotePublications, setRemotePublications] = useState<
    RemoteTrackPublication[]
  >([]);

  const [myRoomName, setMyRoomName] = useState("");
  const [myParticipantName, setMyParticipantName] = useState("");
  // ...
};
export default App;
```

Here's a breakdown of these variables:

- `APPLICATION_SERVER_URL`: This variable stores the URL of the application server. It is used to make HTTP requests to the server.
- `LIVEKIT_URL`: This variable stores the URL of the LiveKit server with which a connection will be established.
- `room`: This variable represents the room object. It is used to manage the video call room.
- `localPublication`: This variable represents the track publication published by the local participant.
- `remotePublications`: This is an array of RemoteTrackPublication objects. It is used to store and manage the tracks publications objects published by remote participants in the video call.
- `mainPublication`: It represents the main video displayed on the page. It can be either the local video publication or one of the remote publications, and it is updated when a user interacts with the application.
- `myRoomName` and `myParticipantName`: These variables are used to store the names of the room and the local participant. They are obtained from user input.

--8<-- "docs/tutorials/shared/configure-urls.md"

<h3 markdown>Configuring the Room events</h3>

In this section, we initialize a new room and configure event handling for various actions within the room.

The `joinRoom()` method is responsible for initializing the room and configuring event handling. It is called when the user clicks the "Join" button on the page. Here's how it's implemented:

```typescript
const App = () => {
  const joinRoom = () => {
    // --- 1) Get a Room object ---
    const room = new Room();
    setRoom(room);

    // --- 2) Specify the actions when events take place in the room ---

    // On every new Track received...
    room.on(
      RoomEvent.TrackSubscribed,
      (_track: RemoteTrack, publication: RemoteTrackPublication) => {
        // Store the new publication in remotePublications array
        setRemotePublications((prevPublications) => [
          ...prevPublications,
          publication,
        ]);
      }
    );

    // On every track destroyed...
    room.on(
      RoomEvent.TrackUnsubscribed,
      (_track: RemoteTrack, publication: RemoteTrackPublication) => {
        // Remove the publication from 'remotePublications' array
        deleteRemoteTrackPublication(publication);
      }
    );

    // Additional event handling can be added here...
  };
  // ...
};
```

This code block performs the following actions:

1. `Initializing the Room`: A new room is initialized using the `new Room()` method. This creates a new room object that can be used to manage the video call.

2. `Configuring Room events`: Event handling is configured for different scenarios within the room. These events are fired when new tracks are subscribed to and when existing tracks are unsubscribed.

   - **`RoomEvent.TrackSubscribed`**: This event is triggered when a new track is received in the room. It manages the storage of the new track publication in the `remotePublications` array. These stored publications are used to display remote tracks on the HTML view, as shown below:

   ```html
   { remotePublications.map((publication) => (
   <div key="{publication.trackSid}">
     {publication.videoTrack && <OvVideo track="{publication.videoTrack}" />}
     {publication.audioTrack && <OvAudio track="{publication.audioTrack}" />}
   </div>
   )); }
   ```

   Here, iterating over the `remotePublications` array allows us to display the video and audio tracks associated with each publication using the _OvVideo_ and _OvAudio_ components respectively.

   - **`RoomEvent.TrackUnsubscribed`**: This event occurs when a track is destroyed, and it takes care of removing the track publication from the `remotePublications` array.

These event handlers are essential for managing the behavior of tracks within the video call. You can further extend the event handling as needed for your application.

!!! info "Take a look at all events"

    You can take a look at all the events in the [Livekit Documentation](https://docs.livekit.io/client-sdk-js/enums/RoomEvent.html)

---

<h3 markdown>Getting a Room token</h3>

Before joining the session, you'll need a valid access token, which allows you to access the room. To obtain this token, a request is made to the **server application**.

The `myRoomName` and `myParticipantName` variables are essential parameters for obtaining a token, as they specify the Livekit room for which you need a token.

Here's how this is implemented in the code:

```typescript
const App = () => {
  const joinRoom = () => {
    // ...

    getToken(myRoomName, myParticipantName).then(async (token: string) => {});
  };
  // ...

  const getToken = useMemo(
    () =>
      async (roomName: string, participantName: string): Promise<string> => {
        try {
          const response = await axios.post(
            APPLICATION_SERVER_URL + "token",
            { roomName, participantName },
            {
              headers: { "Content-Type": "application/json" },
              responseType: "text",
            }
          );
          return response.data;
        } catch (error) {
          // Handle errors here
          console.error("Error getting token:", error);
          throw error;
        }
      },
    [APPLICATION_SERVER_URL]
  );
};
```

This code segment is responsible for retrieving a token from the application server. The tutorial utilizes [axios](https://axios-http.com/){:target="\_blank"} to make the required HTTP requests to obtain the token.

---

<h3 markdown>Connecting to the Room</h3>

The final step involves connecting to the room using the obtained access token and publishing your webcam. Let's examine this part of the code:

```typescript
const App = () => {
  const joinRoom = () => {
    // ...

    // --- 3) Connect to the room with a valid access token ---

    // Get a token from the application backend
    getToken(myRoomName, myParticipantName).then(async (token: string) => {
      // First param is the LiveKit server URL. Second param is the access token
      try {
        await room.connect(LIVEKIT_URL, token);
        // --- 4) Publish your local tracks ---
        await room.localParticipant.setMicrophoneEnabled(true);
        const videoPublication = await room.localParticipant.setCameraEnabled(
          true
        );

        // Set the main video in the page to display our webcam and store our localPublication
        setLocalPublication(videoPublication);
        setMyMainPublication(videoPublication);
      } catch (error) {
        console.log(
          "There was an error connecting to the room:",
          error.code,
          error.message
        );
      }
    });
  };
};
```

Here's a breakdown of this code:

1. After obtaining the access token, the `room.connect()` method is called with the LiveKit server URL and the access token as parameters. This connects to the room. Upon successful connection, the following steps are executed:

   - The local microphone is enabled using `room.localParticipant.setMicrophoneEnabled(true)`.
   - The local camera is enabled using `room.localParticipant.setCameraEnabled(true)`.
   - The local video publication is stored in the `localPublication` variable. This publication is used to display the local video on the page using the _OvVideo_.
   - The local video publication is set as the `mainPublication` variable, which is used to display the main video on the page.

The final code segment successfully connects to the room and sets up the local tracks, allowing you to participate in the video call.

---

<h3 markdown>Leaving the room</h3>

The `leaveRoom()` method is responsible for disconnecting from the room and clearing associated properties.
It is called when the user clicks the "Leave" button on the page. Here's how it's implemented:

```typescript
const leaveRoom = () => {
  // --- 5) Leave the room by calling 'disconnect' method over the Room object ---

  if (room) {
    room.disconnect();
  }

  // Empty all properties...
  setRemotePublications([]);
  setLocalPublication(undefined);
  setRoom(new Room());
};
```

---

<h3 markdown> Displaying Track Publications </h3>

To showcase track publications within the HTML view, we utilize the _OvVideo_ and _OvAudio_ components, which are defined in the `OvVideo.tss` and `OvAudio.tsx` files, respectively.

These components are responsible for presenting video and audio tracks, and you can integrate them into your `App.tsx` file like this:

```html
<div id="video-container" className="col-md-6">
	// Local video
	{localPublication !== undefined ? (
		<div className="stream-container col-md-6 col-xs-6">
			{localPublication.videoTrack && (
				<OvVideo
					onClick={() => handleMainVideoStream(localPublication)}
					track={localPublication.videoTrack}
				/>
			)}
		</div>
	) : null}
	{remotePublications.map((publication) => (
		<div
			key={publication.trackSid}
			className={`stream-container col-md-6 col-xs-6 ${
				publication.kind === 'audio' ? 'hidden' : ''
			}`}
		>
			{publication.videoTrack && (
				<OvVideo
					onClick={() => handleMainVideoStream(publication)}
					track={publication.videoTrack}
				/>
			)}
			{publication.audioTrack && (
				<OvAudio track={publication.audioTrack} />
			)}
		</div>
	))}
</div>
```

Here's what's happening:

<h4>Local Video</h4>

- The `localPublication` variable is used to display the local video. This variable is set when the user joins the room.
- Audio isn't added to the local publication since there's no need to hear one's own audio.
- The `handleMainVideoStream()` method is called when the user clicks on the local video. This method updates the `mainPublication` variable, which is used to display the main video on the page.

<h4>Remote Videos</h4>

- The `remotePublications` array is used to display the remote videos. This array is updated when a new track is subscribed to or when an existing track is unsubscribed. Iterating over this array allows us to display the video and audio tracks associated with each publication using the _OvVideo_ and _OvAudio_ components respectively. The code also hides the audio tracks using the `hidden` class because we don't want to display them.

- The `handleMainVideoStream()` method is called when the user clicks on a remote video. This method updates the `mainPublication` variable, which is used to display the main video on the page.

Both the audio and video components receive a track, which is then attached to the corresponding HTML elements (`<audio>` and `<video>`). For example, here's the implementation of the _OvVideo_:

```javascript
interface OvVideoProps {
  track: LocalVideoTrack | RemoteVideoTrack;
  onClick?: () => void;
}

const OvVideo: FC<OvVideoProps> = ({ track, onClick }) => {
  const videoRef: React.MutableRefObject<null | HTMLVideoElement> =
    useRef(null);

  useEffect(() => {
    if (videoRef.current) {
      track.attach(videoRef.current);
    }
  }, [track]);

  return <video onClick={onClick} ref={videoRef} playsInline autoPlay={true} />;
};

export default OvVideo;
```

The `track` property is used to attach the track to the `<video>` element. This is done using the `attach()` method, which is available on both `LocalVideoTrack` and `RemoteVideoTrack` objects.

---

## Deploying openvidu-react (TODO)

<h3> 1) Build the docker image</h3>

Under the root project folder, you can see the `openvidu-react/docker/` directory. Here it is included all the required files yo make it possible the deployment with OpenVidu.

First of all, you will need to create the **openvidu-react** docker image. Under `openvidu-react/docker/` directory you will find the `create_image.sh` script. This script will create the docker image with the [openvidu-basic-node](../application-server/node.md) as application server and the static files.

```bash
./create_image.sh openvidu/openvidu-react-demo:X.Y.Z
```

This script will create an image named `openvidu/openvidu-react-demo:X.Y.Z`. This name will be used in the next step.

<h3> 2) Deploy the docker image</h3>

Time to deploy the docker image. You can follow the [Deploy OpenVidu based application with Docker](#) guide for doing this.
