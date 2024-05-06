# openvidu-js

[Source code :simple-github:](https://github.com/OpenVidu/openvidu-tutorials){ .md-button target=\_blank }

This tutorial is a simple video-call application built with plain JavaScript, HTML and CSS. It is a good starting point to learn how to use Livekit in your web applications.

## Running this tutorial

Running this tutorial is straightforward, and here's what you'll need:

<h3>1. OpenVidu Server Installation</h3>

--8<-- "docs/tutorials/shared/run-openvidu-dev.md"

<h3>2. Start your preferred server application sample</h3>

--8<-- "docs/tutorials/shared/application-server-tabs.md"

<h3>3. Launch the client application tutorial</h3>

To run the client application tutorial, you'll need a HTTP web server installed on your development computer. If you have Node.js installed, you can easily set up [http-server](https://github.com/indexzero/http-server){:target="\_blank"}. Here's how to install it:

```bash
npm install --location=global http-server
```

After installing http-server, serve the tutorial by executing the following command:

```bash
# Using the same repository openvidu-livekit-tutorials from step 2
http-server -p 5080 openvidu-tutorials/openvidu-js/web
```

Once the server is up and running, you can test the application by visiting [`http://localhost:5080`](http://localhost:5080){:target="\_blank"}. You should see a screen like this:

<div class="grid-container">

<div class="grid-50"><p><a class="glightbox" href="../../../../assets/images/insecure-join.png" data-type="image" data-width="100%" data-height="auto" data-desc-position="bottom"><img src="../../../../assets/images/insecure-join.png" loading="lazy"/></a></p></div>

<div class="grid-50"><p><a class="glightbox" href="../../../../assets/images/insecure-session.png" data-type="image" data-width="100%" data-height="auto" data-desc-position="bottom"><img src="../../../../assets/images/insecure-session.png" loading="lazy"/></a></p></div>

</div>

--8<-- "docs/tutorials/shared/testing-other-devices.md"

## Understanding the code

This application is designed to be beginner-friendly and consists of only three essential files:

- `app.js`: This is the main JavaScript file for the sample application. It utilizes the livekit-client library to connect to video rooms. You have the flexibility to modify this file according to your specific requirements.
- `style.css`: This file contains CSS classes that can be used to style the index.html page. You can customize the styles to match your preferences.
- `index.html`: This HTML file is responsible for creating the user interface. It contains the form to connect to a video call and the video call itself. You can customize this file to adapt it to your needs. It also includes two script tags linking to the following JavaScript files:

---

<h3 markdown>Using `livekit-client`</h3>

```html
<script src="https://cdn.jsdelivr.net/npm/livekit-client/dist/livekit-client.umd.min.js"></script>
<!-- (1)! -->
<script src="app.js"></script>
```

1. This line imports the livekit-client latest. If you need to use a specific version, add it to the URL. For example, to use version 1.10.0, use `https://cdn.jsdelivr.net/npm/livekit-client@1.10.0/dist/livekit-client.umd.min.js`.

Let's take a closer look at how `app.js` leverages the `livekit-client` to establish a connection to a video room.

<h3 markdown>Defining variables</h3>

The first two variables in the code are `APPLICATION_SERVER_URL` and `LIVEKIT_URL`. These variables are used to configure the URLs for the application server and Livekit server, respectively.

--8<-- "docs/tutorials/shared/configure-urls.md"

```javascript
// For local development, leave these variables empty
// For production, configure them with correct URLs depending on your deployment
var APPLICATION_SERVER_URL = '';
var LIVEKIT_URL = '';

// If APPLICATION_SERVER_URL is not configured, use default value from local development
if (!APPLICATION_SERVER_URL) {
	if (window.location.hostname === 'localhost') {
		APPLICATION_SERVER_URL = 'http://localhost:6080/';
	} else {
		APPLICATION_SERVER_URL = 'https://' + window.location.hostname + ':6443/';
	}
}

// If LIVEKIT_URL is not configured, use default value from local development
if (!LIVEKIT_URL) {
	if (window.location.hostname === 'localhost') {
		LIVEKIT_URL = 'ws://localhost:7880/';	
	} else {
		LIVEKIT_URL = 'wss://' + window.location.hostname + ':7443/';
	}
}
```

Besides, the following peace of code declare the variables that will be utilized at various points throughout the script:

```javascript
var LivekitClient = window.LivekitClient;
var room;
```

- `LivekitClient` is the LivekitClient object, serving as the entry point to the library.

- `room` will represent the video call that we are going to connect to.

Further, within the `joinSession` method, we initialize two parameters with values retrieved from the HTML inputs:

```javascript
function joinRoom() {
	var myRoomName = document.getElementById('roomName').value;
	var myUserName = document.getElementById('userName').value;

	// ...
}
```

<h3 markdown>Configuring the Room events</h3>

In this section, we initialize a new room and configure event handling for various actions within the room. Let's break it down step by step:

```javascript
function joinRoom() {
	// ...

	// --- 1) Get a Room object ---

	room = new LivekitClient.Room();

	// --- 2) Specify the actions when events take place in the room ---

	// On every new Track received...
	room.on(
		LivekitClient.RoomEvent.TrackSubscribed,
		(track, publication, participant) => {
			const element = track.attach();
			element.id = track.sid;
			element.className = 'removable';
			document.getElementById('video-container').appendChild(element);
			if (track.kind === 'video') {
				appendUserData(element, participant.identity);
			}
		}
	);

	// On every new Track destroyed...
	room.on(
		LivekitClient.RoomEvent.TrackUnsubscribed,
		(track, publication, participant) => {
			track.detach();
			document.getElementById(track.sid)?.remove();
			if (track.kind === 'video') {
				removeUserData(participant);
			}
		}
	);

	// Additional event handling can be added here...
}
```

This code block performs the following actions:

1. It creates a new `Room` object using `LivekitClient.Room()`. This object represents the video call room.

2. Event handling is configured for different scenarios within the room. These events are fired when new tracks are subscribed to and when existing tracks are unsubscribed.

   - **`LivekitClient.RoomEvent.TrackSubscribed`**: This event is triggered when a new track is received in the room. It handles the attachment of the track to the HTML page, assigning an ID, and appending it to the "video-container" element. If the track is of kind `video`, user data is appended as well.

   - **`LivekitClient.RoomEvent.TrackUnsubscribed`**: This event occurs when a track is destroyed, and it takes care of detaching the track from the HTML page and removing it from the DOM. If the track is a `video` track, user data is also removed.

These event handlers are essential for managing the behavior of tracks within the video call. You can further extend the event handling as needed for your application.

!!! info "Take a look at all events"

    You can take a look at all the events in the [Livekit Documentation](https://docs.livekit.io/client-sdk-js/enums/RoomEvent.html)

---

<h3 markdown>Getting a Room token</h3>

Before joining the session, you'll need a valid access token, which allows you to access the room. To obtain this token, a request is made to the **server application**.

The `myRoomName` and `myUserName` variables are essential parameters for obtaining a token, as they specify the Livekit room for which you need a token.

Here's how this is implemented in the code:

```javascript
function joinRoom() {
	// ...

	// --- 3) Connect to the room with a valid access token ---

	// Get a token from the application backend
	getToken(myRoomName, myUserName).then((token) => {
		// See next point to see how to connect to the session using 'token'
	});
}

function getToken(roomName, participantName) {
	return new Promise((resolve, reject) => {
		$.ajax({
			type: 'POST',
			url: APPLICATION_SERVER_URL + 'token',
			data: JSON.stringify({
				roomName,
				participantName,
			}),
			headers: { 'Content-Type': 'application/json' },
			success: (token) => resolve(token),
			error: (error) => reject(error),
		});
	});
}
```

This code segment is responsible for retrieving a token from the application server. The tutorial utilizes the `jQuery.ajax()` method to make the required HTTP requests to obtain the token.

---

<h3 markdown>Connecting to the room</h3>

The final step involves connecting to the room using the obtained access token and publishing your webcam. Let's examine this part of the code:

```javascript
function joinRoom() {
	// ...

	// --- 3) Connect to the room with a valid access token ---

	// Get a token from the application backend
	getToken(myRoomName, myUserName).then((token) => {
		// First param is the LiveKit server URL. Second param is the access token
		room
			.connect(LIVEKIT_URL, token)
			.then(() => {
				// --- 4) Set page layout for active call ---

				document.getElementById('room-title').innerText = myRoomName;
				document.getElementById('join').style.display = 'none';
				document.getElementById('room').style.display = 'block';

				// --- 5) Publish your local tracks ---

				room.localParticipant.setMicrophoneEnabled(true);
				room.localParticipant.setCameraEnabled(true).then((publication) => {
					const element = publication.track.attach();
					document.getElementById('video-container').appendChild(element);
					initMainVideo(element, myUserName);
					appendUserData(element, myUserName);
					element.className = 'removable';
				});
			})
			.catch((error) => {
				console.log(
					'There was an error connecting to the room:',
					error.code,
					error.message
				);
			});
	});
}
```

Here's a breakdown of this code:

1. After obtaining the access token, the `room.connect()` method is called with the LiveKit server URL and the access token as parameters. This connects to the room. Upon successful connection, the following steps are executed:

   - The page layout is adjusted to reflect the active call. The room title is set, and the "join" button is hidden, while the "room" elements are displayed.

   - Local tracks, such as the microphone and camera, are enabled using room.localParticipant.setMicrophoneEnabled(true) and room.localParticipant.setCameraEnabled(true).

   - The local video track is attached to an HTML element, appended to the "video-container," and user data is added to it.

The final code segment successfully connects to the room and sets up the local tracks, allowing you to participate in the video call.

---

<h3 markdown> Leaving the room </h3>

Whenever we want a user to leave the room, we just need to call `room.disconnect` method. We also make sure to call the method before the page is unloaded using event `window.onbeforeunload`.

```javascript
function leaveRoom() {
	// --- 6) Leave the room by calling 'disconnect' method over the Room object ---

	room.disconnect();

	// Removing all HTML elements with user's nicknames.
	// HTML videos are automatically removed when leaving a Room
	removeAllUserData();

	// Back to 'Join room' page
	document.getElementById('join').style.display = 'block';
	document.getElementById('room').style.display = 'none';
}

window.onbeforeunload = function () {
	if (room) room.disconnect();
};
```

## Deploying openvidu-js (TODO)

> This tutorial image is **officially released for OpenVidu** under the name `openvidu/openvidu-js-demo:X.Y.Z` so you do not need to build it by yourself. However, if you want to deploy a custom version of openvidu-js, you will need to build a new one. You can keep reading for more information.

<h3> 1. Build the docker image</h3>

Under the root project folder, you can see the `openvidu-js/docker/` directory. Here it is included all the required files yo make it possible the deployment with OpenVidu.

First of all, you will need to create the **openvidu-js** docker image. Under `openvidu-js/docker/` directory you will find the `create_image.sh` script. This script will create the docker image with the [openvidu-basic-node](../application-server/nodejs.md) as application server and the static files.

```bash
./create_image.sh openvidu/openvidu-js-demo:X.Y.Z
```

This script will create an image named `openvidu/openvidu-js-demo:X.Y.Z`. This name will be used in the next step.

<h3> 2. Deploy the docker image </h3>

Time to deploy the docker image. You can follow the [Deploy OpenVidu based application with Docker](#) guide for doing this.
