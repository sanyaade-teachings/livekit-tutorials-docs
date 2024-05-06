# openvidu-vue

[Source code :simple-github:](https://github.com/OpenVidu/openvidu-livekit-tutorials){ .md-button target=\_blank }

This tutorial is a simple video-call application built with [Vue.js](https://vuejs.org/){:target="\_blank"} and [Livekit JavaScript SDK](https://docs.livekit.io/client-sdk-js/){:target="\_blank"}. It is based on the [Livekit JavaScript tutorial](./javascript.md) but using Vue.js framework instead of plain web technologies.

## Running this tutorial

Running this tutorial is straightforward, and here's what you'll need:

<h3>1. OpenVidu Server Installation</h3>

--8<-- "docs/tutorials/shared/run-openvidu-dev.md"

<h3>2. Start your preferred server application sample</h3>

--8<-- "docs/tutorials/shared/application-server-tabs.md"

<h3>3. Launch the client application tutorial</h3>

To run the client application tutorial, you'll need [NPM](https://docs.npmjs.com/downloading-and-installing-node-js-and-npm){:target="\_blank"} installed on your development computer.

Check if you have installed it by running the following command:

```bash
npm -v
```

Once you've confirmed that NPM is installed, you can proceed with the tutorial by following these steps:

```bash
# Assuming you've already cloned the 'openvidu-livekit-tutorials' repository as described in Step 2

cd openvidu-livekit-tutorials/openvidu-vue
npm install
npm run serve
```

Once the server is up and running, you can test the application by visiting [`http://localhost:5080`](http://localhost:5080){:target="\_blank"}. You should see a screen like this:

<div class="grid-container">

<div class="grid-50"><p><a class="glightbox" href="../../../../assets/images/insecure-join.png" data-type="image" data-width="100%" data-height="auto" data-desc-position="bottom"><img src="../../../../assets/images/insecure-join.png" loading="lazy"/></a></p></div>

<div class="grid-50"><p><a class="glightbox" href="../../../../assets/images/insecure-session.png" data-type="image" data-width="100%" data-height="auto" data-desc-position="bottom"><img src="../../../../assets/images/insecure-session.png" loading="lazy"/></a></p></div>

</div>

--8<-- "docs/tutorials/shared/testing-other-devices.md"

## Understanding the code

This Vue project has been generated using the Vue CLI tool. You may come across various configuration files and other items that aren't essential for our tutorial. Our focus will be on the key files located within the `src/` and `src/components` directory:

- `App.vue`: This file defines the main application component and its HTML template. It's responsible for handling tasks such as joining a video call and managing the video calls themselves.

- `OvVideo.vue`: This file introduces the `OvVideo` component. This component is responsible for encapsulating the `<video>` element that displays the video track within the application.
- `OvAudio.vue`: This file introduces the `OvAudio` component. This component is responsible for encapsulating the `<audio>` element that displays the audio track within the application.

---

<h3 markdown>Using `livekit-client`</h3>

To understand how the `livekit-client` NPM package is used in `App.vue`, let's break it down step by step. The code in this section provides an overview of how this package is utilized.

In the beginning, we import several essential objects from the `livekit-client` package that will be used throughout the code. These objects include:

```javascript
import OvVideo from './components/OvVideo.vue';
import OvAudio from './components/OvAudio.vue';
import { Room, RoomEvent } from 'livekit-client';
```

These imported objects are crucial for establishing and managing video calls. They represent different components and events associated with the video call system. We'll use them in subsequent parts of our code to create and manage video calls.

<h3 markdown>Defining variables</h3>

After importing the required objects, we define a few variables that will be used later in the code:

```javascript
// For local development, leave these variables empty
// For production, configure them with correct URLs depending on your deployment
let APPLICATION_SERVER_URL = ''
let LIVEKIT_URL = ''

// If APPLICATION_SERVER_URL is not configured, use default value from local development
if (!APPLICATION_SERVER_URL) {
	if (window.location.hostname === 'localhost') {
		APPLICATION_SERVER_URL = 'http://localhost:6080/'
	} else {
		APPLICATION_SERVER_URL = 'https://' + window.location.hostname + ':6443/'
	}
}

// If LIVEKIT_URL is not configured, use default value from local development
if (!LIVEKIT_URL) {
	if (window.location.hostname === 'localhost') {
		LIVEKIT_URL = 'ws://localhost:7880/'
	} else {
		LIVEKIT_URL = 'wss://' + window.location.hostname + ':7443/'
	}
}

export default {
	name: 'App',

	components: {
		OvVideo,
		OvAudio,
	},

	data() {
		return {
			// OpenVidu objects
			room: undefined,
			mainPublication: undefined,
			localPublication: undefined,
			remotePublications: [],

			// Join form
			myRoomName: 'RoomA',
			myParticipantName: 'Participant' + Math.floor(Math.random() * 100),
		};
	},

	// ...
};
```

Here's a breakdown of these variables:

- `APPLICATION_SERVER_URL`: This variable stores the URL of the application server. It is used to make HTTP requests to the server.
- `LIVEKIT_URL`: This variable stores the URL of the LiveKit server with which a connection will be established.
- `room`: This variable will be used to manage the current video call room.
- `localPublication`: This variable represents the track publication published by the local participant.
- `remotePublications`: This is an array of RemoteTrackPublication objects. It is used to store and manage the tracks publications objects published by remote participants in the video call.
- `myRoomName` and `myParticipantName`: These variables are used to store the names of the room and the local participant. They are obtained from user input.
- `mainPublication`: It represents the main video displayed on the page. It can be either the local video publication or one of the remote publications, and it is updated when a user interacts with the application.

--8<-- "docs/tutorials/shared/configure-urls.md"

<h3 markdown>Configuring the Room events</h3>

In this section, we initialize a new room and configure event handling for various actions within the room.

The `joinRoom()` method is responsible for initializing the room and configuring event handling. It is called when the user clicks the "Join" button on the page. Here's how it's implemented:

```javascript
export default {
	// ...

	methods: {
		joinRoom() {
			// --- 1) Init a room ---
			this.room = new Room();

			// --- 2) Specify the actions when events take place in the room ---

			// On every new Track received...
			this.room.on(
				RoomEvent.TrackSubscribed,
				(track, publication, participant) => {
					console.log('Track subscribed', track, publication, participant);
					// Store the new publication in remotePublications array
					this.remotePublications.push(publication);
				}
			);

			// On every track destroyed...
			this.room.on(
				RoomEvent.TrackUnsubscribed,
				(track, publication, participant) => {
					console.log('Track unsubscribed', track, publication, participant);
					// Remove the publication from 'remotePublications' array
					this.deleteRemoteTrackPublication(publication);
				}
			);

			// Additional event handling can be added here...
		},
	},
};
```

This code block performs the following actions:

1. `Initializing the Room`: A new room is initialized using the `new Room()` method. This creates a new room object that can be used to manage the video call.

2. `Configuring Room events`: Event handling is configured for different scenarios within the room. These events are fired when new tracks are subscribed to and when existing tracks are unsubscribed.

   - **`RoomEvent.TrackSubscribed`**: This event is triggered when a new track is received in the room. It manages the storage of the new track publication in the `remotePublications` array. These stored publications are used to display remote tracks on the HTML view, often with the help of Vue's `v-for` directive as shown below:

   ```html
   <div v-for="publication in remotePublications" :key="publication.trackSid">
   	<OvVideo :track="publication.videoTrack" />
   	<OvAudio :track="publication.audioTrack" />
   </div>
   ```

   Here, using the `v-for` directive, we iterate over the `remotePublications` array and display the video and audio tracks associated with each publication using the _OvVideo_ and _OvAudio_ components respectively.

- **`RoomEvent.TrackUnsubscribed`**: This event occurs when a track is destroyed, and it takes care of removing the track publication from the `remotePublications` array.

These event handlers are essential for managing the behavior of tracks within the video call. You can further extend the event handling as needed for your application.

!!! info "Take a look at all events"

    You can take a look at all the events in the [Livekit Documentation](https://docs.livekit.io/client-sdk-js/enums/RoomEvent.html)

---

<h3 markdown>Getting a Room token</h3>

Before joining the session, you'll need a valid access token, which allows you to access the room. To obtain this token, a request is made to the **server application**.

The `myRoomName` and `myParticipantName` variables are essential parameters for obtaining a token, as they specify the Livekit room for which you need a token.

Here's how this is implemented in the code:

```javascript
export default {
	// ...

	methods: {
		joinRoom() {
			// ...

			// --- 3) Connect to the room with a valid access token ---

			// Get a token from the application backend
			this.getToken(this.myRoomName, this.myParticipantName).then((token) => {
				// See next point to see how to connect to the session using 'token'
			});
		},

		async getToken(roomName, participantName) {
			try {
				const response = await axios.post(
					APPLICATION_SERVER_URL + 'token',
					{ roomName, participantName },
					{
						headers: { 'Content-Type': 'application/json' },
						responseType: 'text',
					}
				);
				return response.data;
			} catch (error) {
				// Handle errors here
				console.error('Error getting token:', error);
				throw error;
			}
		},
	},
};
```

This code segment is responsible for retrieving a token from the application server. The tutorial utilizes [axios](https://axios-http.com/){:target="\_blank"} to make the required HTTP requests to obtain the token.

---

<h3 markdown>Connecting to the room</h3>

The final step involves connecting to the room using the obtained access token and publishing your webcam. Let's examine this part of the code:

```javascript
export default {
	// ...

	methods: {
		joinRoom() {
			// ...

			// --- 3) Connect to the room with a valid access token ---

			// Get a token from the application backend
			this.getToken(this.myRoomName, this.myParticipantName).then(
				async (token) => {
					// First param is the LiveKit server URL. Second param is the access token
					try {
						await this.room.connect(LIVEKIT_URL, token);
						// --- 4) Publish your local tracks ---
						await this.room.localParticipant.setMicrophoneEnabled(true);
						const videoPublication =
							await this.room.localParticipant.setCameraEnabled(true);

						// Set the main video in the page to display our webcam and store our localPublication
						this.localPublication = videoPublication;
						this.mainPublication = videoPublication;
					} catch (error) {
						console.log(
							'There was an error connecting to the room:',
							error.code,
							error.message
						);
					}
				}
			);
		},
	},
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

```javascript
export default {
	// ...

	methods: {
		leaveRoom() {
			// --- 5) Leave the session by calling 'disconnect' method over the Room object ---

			if (this.room) {
				this.room.disconnect();
			}

			// Empty all properties...
			this.room = undefined;
			this.mainPublication = undefined;
			this.localPublication = undefined;
			this.remotePublications = [];

			// Remove beforeunload listener
			window.removeEventListener('beforeunload', this.leaveRoom);
		},
	},
};
```

We also configure the `beforeunload` event to call the `leaveRoom()` method when the user closes the browser window or tab. This ensures that the user leaves the room when they close the browser window or tab.

```javascript
export default {
	// ...

	methods: {
		joinRoom() {
			// ...

			window.addEventListener('beforeunload', this.leaveRoom);
		},
	},
};
```

---

<h3 markdown> Displaying Track Publications </h3>

To showcase track publications within the HTML view, we utilize the _OvVideo_ and _OvAudio_ components, which are defined in the `OvVideo.vue` and `OvAudio.vue` files respectively.

These components are responsible for presenting video and audio tracks, and you can integrate them into your `App.vue` file as shown below:

```html
<!-- Local video -->
<div
	v-if="localPublication && localPublication.videoTrack"
	id="video-container"
	class="col-md-6"
>
	<p class="participant-name">{{ myParticipantName }}</p>
	<OvVideo
		:track="localPublication.videoTrack"
		@click="updateMainPublication(localPublication)"
	/>
</div>

<!-- Remote videos -->
<div v-for="publication in remotePublications" :key="publication.trackSid">
	<p v-if="publication.videoTrack" class="participant-name">
		{{ getParticipantName(publication.trackSid) }}
	</p>

	<OvVideo
		v-if="publication.videoTrack"
		:track="publication.videoTrack"
		@click="updateMainPublication(publication)"
	/>
	<OvAudio v-if="publication.audioTrack" :track="publication.audioTrack" />
</div>
```

Here's what's happening:

<h4>Local Video</h4>

- We use the `v-if` directive to conditionally display the local video, ensuring it's only visible when a user has joined the room.
- Audio isn't added to the local publication since there's no need to hear one's own audio.
- The `updateMainStreamManager()` method is called when the user clicks on the local video. This method updates the `mainPublication` variable, which is used to display the main video on the page.

<h4>Remote Videos</h4>

- We use the `v-for` directive to iterate over the `remotePublications` array and display the video and audio tracks associated with each publication using the _OvVideo_ and _OvAudio_ respectively. The code also hides the audio tracks using the `hidden` class because we don't want to display them.
- The `updateMainStreamManager()` method is called when the user clicks on a remote video. This method updates the `mainPublication` variable, which is used to display the main video on the page.

Both the audio and video components receive a track, which is then attached to the corresponding HTML elements (`<audio>` and `<video>`). For example, here's the implementation of the _OvVideo_:

```javascript
<template>
	<video autoplay/>
</template>

<script>
export default {
	name: 'OvVideo',

	props: {
		track: Object,
	},

	mounted () {
		this.track.attach(this.$el);
	},
};
</script>
```

The `track` property is used to attach the track to the `<video>` element. This is done using the `attach()` method, which is available on both `LocalVideoTrack` and `RemoteVideoTrack` objects.

---

## Deploying openvidu-vue (TODO)

<h3> 1) Build the docker image</h3>

Under the root project folder, you can see the `openvidu-vue/docker/` directory. Here it is included all the required files yo make it possible the deployment with OpenVidu.

First of all, you will need to create the **openvidu-vue** docker image. Under `openvidu-vue/docker/` directory you will find the `create_image.sh` script. This script will create the docker image with the [openvidu-basic-node](../application-server/nodejs.md) as application server and the static files.

```bash
./create_image.sh openvidu/openvidu-vue-demo:X.Y.Z
```

This script will create an image named `openvidu/openvidu-vue-demo:X.Y.Z`. This name will be used in the next step.

<h3> 2) Deploy the docker image</h3>

Time to deploy the docker image. You can follow the [Deploy OpenVidu based application with Docker](/deployment/deploying-openvidu-apps/#with-docker) guide for doing this.
