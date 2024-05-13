# openvidu-electron

[Source code :simple-github:](https://github.com/OpenVidu/openvidu-livekit-tutorials){ .md-button target=\_blank }

This tutorial shows how to build an **Electron** application using Livekit. It can be compiled into a native desktop application for _Windows_, _OSX_ and _Linux_.

## Running this tutorial

Running this tutorial is straightforward, and here's what you'll need:

### 1. OpenVidu Server Installation

--8<-- "docs/tutorials/shared/run-openvidu-dev.md"

### 2. Run a server application

--8<-- "docs/tutorials/shared/application-server-tabs.md"

### 3. Run the client application

To run the client application tutorial, you'll need [NPM](https://docs.npmjs.com/downloading-and-installing-node-js-and-npm){:target="\_blank"} installed on your development computer.

Check if you have installed it by running the following command:

```bash
npm -v
```

Once you've confirmed that NPM is installed, you can proceed with the tutorial by following these steps:

```bash
# Assuming you've already cloned the 'openvidu-livekit-tutorials' repository as described in Step 2

cd openvidu-livekit-tutorials/openvidu-electron
npm install
npm start
```

The application will seamlessly initiate as a native desktop program, adapting itself to the specific operating system you're using. On Windows, it will launch as a Windows application; on macOS, it will function as a macOS application, and on Linux, it will operate as a Linux application.

<div class="grid-container">

<div class="grid-50"><p><a class="glightbox" href="../../../../assets/images/openvidu-electron.png" data-type="image" data-width="100%" data-height="auto" data-desc-position="bottom"><img src="../../../../assets/images/openvidu-electron.png" loading="lazy"/></a></p></div>

<div class="grid-50"><p><a class="glightbox" href="../../../../assets/images/openvidu-electron2.png" data-type="image" data-width="100%" data-height="auto" data-desc-position="bottom"><img src="../../../../assets/images/openvidu-electron2.png" loading="lazy"/></a></p></div>

</div>

## Understanding the code

As Electron app, the code is divided into two parts: the **main process** and the **renderer process**.

The most important files are:

- `main.js`: Main process code, responsible for creating the application window and managing the application lifecycle.
- `index.js`: Renderer process code, responsible for the application UI and logic. It uses the Livekit SDK to connect to the Server and manage the videoconference.
- `index.html`: HTML file that contains the application UI.
- `style.css`: CSS file that contains the application styles.
- `modal.html`: HTML file that contains the modal UI for selecting the screen to share when using the screen sharing feature.

We have implemented screen-sharing capabilities in this application because the process is slightly different from the rest of platforms that support it. Electron does not provide a default screen selector dialog, so we must implement it ourselves.

Let's see how the code works:

<h3 markdown>Using `livekit-client`</h3>

To understand how the `livekit-client` NPM package is used in `index.js`, let's break it down step by step. The code in this section provides an overview of how this package is utilized.

In the beginning, we import several essential objects from the `livekit-client` package that will be used throughout the code. These objects include:

```javascript
const { Room, RoomEvent } = require("livekit-client");
```

These imported objects are crucial for establishing and managing video calls. They represent different components and events associated with the video call system. We'll use them in subsequent parts of our code to create and manage video calls.

<h3 markdown>Defining variables</h3>

After importing the required objects, we define a few variables that will be used later in the code:

```javascript
const ipcRenderer = require("electron").ipcRenderer;
const { BrowserWindow } = require("@electron/remote");
var room;
var publisher;
var myParticipantName;
var myRoomName;
var isScreenShared = false;
var screenSharePublication;

// Configure this constants with correct URLs depending on your deployment
const APPLICATION_SERVER_URL = "http://localhost:6080/";
const LIVEKIT_URL = "ws://localhost:7880/";
```

- `ipcRenderer`: This object facilitates communication with the main process. It is used to send and receive messages from the main process and it is used for the screen sharing dialog.
- `BrowserWindow`: It is responsible for creating a new window for the screen sharing dialog. The `modal.html` file is used as the content of this window.
- `room`: Here, we store the Room object used for video call management.
- `publisher`: This variable holds the LocalParticipant object, which manages the local participant.
- `myParticipantName`: It keeps track of the name of the local participant.
- `myRoomName`: This variable stores the name of the room.
- `isScreenShared`: A boolean variable that indicates whether screen sharing is active.
- `screenSharePublication`: Used to manage the screen sharing publication.
- `APPLICATION_SERVER_URL`: This variable stores the URL of the application server. It is used to make HTTP requests to the server.
- `LIVEKIT_URL`: This variable stores the URL of the LiveKit server with which a connection will be established.

!!! warning "Configure the URLs"

    You should configure `APPLICATION_SERVER_URL` and `LIVEKIT_URL` constants with the correct URLs depending on your deployment.

<h3 markdown>Configuring the Room events</h3>

In this section, we initialize a new room and configure event handling for various actions within the room. Let's break it down step by step:

```javascript
async function joinRoom() {
  myRoomName = document.getElementById("roomName").value;
  myParticipantName = document.getElementById("participantName").value;

  // --- 1) Get a Room object ---
  room = new Room();

  // --- 2) Specify the actions when events take place in the room ---

  // On every new Track received...
  room.on(RoomEvent.TrackSubscribed, (track, publication, participant) => {
    const element = track.attach();
    element.id = track.sid;
    element.className = "removable";
    document.getElementById("remote-participants").appendChild(element);
  });

  // On every new Track destroyed...
  room.on(RoomEvent.TrackUnsubscribed, (track, publication, participant) => {
    track.detach();
    document.getElementById(track.sid)?.remove();
  });

  // Additional event handling can be added here...
}
```

This code block performs the following actions:

1. It creates a new `Room` object using `Room()`. This object represents the video call room.

2. Event handling is configured for different scenarios within the room. These events are fired when new tracks are subscribed to and when existing tracks are unsubscribed.

   - **`RoomEvent.TrackSubscribed`**: This event is triggered when a new track is received in the room. It handles the attachment of the track to the HTML page, assigning an ID, and appending it to the "**remote-participants**" element.

   - **`RoomEvent.TrackUnsubscribed`**: This event occurs when a track is destroyed, and it takes care of detaching the track from the HTML page and removing it from the DOM.

These event handlers are essential for managing the behavior of tracks within the video call. You can further extend the event handling as needed for your application.

!!! info "Take a look at all events"

    You can take a look at all the events in the [Livekit Documentation](https://docs.livekit.io/client-sdk-js/enums/RoomEvent.html)

---

<h3 markdown>Getting a Room token</h3>

Before joining the session, you'll need a valid access token, which allows you to access the room. To obtain this token, a request is made to the **server application**.

The `myRoomName` and `myParticipantName` variables are essential parameters for obtaining a token, as they specify the Livekit room for which you need a token.

Here's how this is implemented in the code:

```javascript
async function joinRoom() {
  // ...

  // Get a token from the application backend
  const token = await getToken(myRoomName, myParticipantName);
  // See next point to see how to connect to the session using 'token'

  // ...
}

async function getToken(roomName, participantName) {
  try {
    const response = await axios.post(
      APPLICATION_SERVER_URL + "token",
      {
        roomName,
        participantName,
      },
      {
        headers: {
          "Content-Type": "application/json",
        },
      }
    );

    return response.data;
  } catch (error) {
    console.error("No connection to application server", error);
  }
}
```

This code segment is responsible for retrieving a token from the application server. The tutorial utilizes [axios](https://axios-http.com/){:target="\_blank"} to make the required HTTP requests to obtain the token.

---

<h3 markdown>Connecting to the room</h3>

The final step involves connecting to the room using the obtained access token and publishing your webcam. Let's examine this part of the code:

```javascript
async function getToken(roomName, participantName) {
  // ...

  // Get a token from the application backend
  const token = await getToken(myRoomName, myParticipantName);

  await room.connect(LIVEKIT_URL, token);

  showRoom();
  // --- 4) Publish your local tracks ---
  await room.localParticipant.setMicrophoneEnabled(true);
  const publication = await room.localParticipant.setCameraEnabled(true);
  const element = publication.track.attach();
  element.className = "removable";
  document.getElementById("local-participant").appendChild(element);
}
```

Here's a breakdown of this code:

1. After obtaining the access token, the `room.connect()` method is called with the LiveKit server URL and the access token as parameters. This connects to the room. Upon successful connection, the following steps are executed:

   - The local microphone is enabled using `room.localParticipant.setMicrophoneEnabled(true)`.
   - The local camera is enabled using `room.localParticipant.setCameraEnabled(true)`.
   - The local publication is added to the HTML page using `publication.track.attach()`.
   - The local publication is appended to the "**local-participant**" element.

The final code segment successfully connects to the room and sets up the local tracks, allowing you to participate in the video call.

---

<h3 markdown>Leaving the room</h3>

The `leaveRoom()` method is responsible for disconnecting from the room and clearing associated properties.
It is called when the user clicks the "Leave" button on the page. Here's how it's implemented:

```javascript
function leaveRoom() {
  // --- 5) Leave the room by calling 'disconnect' method over the Room object ---

  room.disconnect();
  // Remove all HTML elements associated with the participant
  removeAllParticipantElements();
  hideRoom();
}
```

This code segment calls the `room.disconnect()` method to disconnect from the room. It also removes all HTML elements associated with the participant and hides the room.

---

<h3 markdown>Screen sharing</h3>

The process to screen-share is slightly different from the rest of platforms that support it. Let's delve into how it's specifically implemented in this guide.

<h4 markdown>Enabling screen sharing</h4>

Electron does not provide a default screen selector dialog as browsers do, so we must implement it ourselves (that is the purpose of `modal.html` file).

This feature is implemented in this tutorial using the `desktopCapturer` API. This API is used to capture the screen of the desktop. The captured stream is then published to the room.

First of all, the tutorial has a button to toggle screen sharing. When the user clicks on it, the `toggleScreenShare()` method is called:

```javascript
async function toggleScreenShare() {
  const enabled = !isScreenShared;

  if (enabled) {
    openScreenShareModal();
  } else {
    // Disable screen sharing
  }
}
```

This method checks if screen sharing is enabled or not. If it is enabled, it opens the screen sharing dialog. If it is disabled, it disables screen sharing.

The `openScreenShareModal()` method is responsible for opening the screen sharing dialog. Here's how it's implemented:

```javascript
function openScreenShareModal() {
  let win = new BrowserWindow({
    parent: require("@electron/remote").getCurrentWindow(),
    modal: true,
    minimizable: false,
    maximizable: false,
    webPreferences: {
      nodeIntegration: true,
      enableRemoteModule: true,
      contextIsolation: false,
    },
    resizable: false,
  });
  require("@electron/remote")
    .require("@electron/remote/main")
    .enable(win.webContents);

  win.setMenu(null);

  var theUrl = "file://" + __dirname + "/modal.html";
  win.loadURL(theUrl);
}
```

This method creates a new window using the `BrowserWindow` object abd loads the `modal.html` file into it, which contains the screen sharing dialog.

Let's take a look at the `modal.html` file:

```javascript
// Define arrays to store available screens and HTML elements
var availableScreens = [];
var htmlElements = [];
var selectedElement;

// Import required Electron modules
const desktopCapturer = require("@electron/remote").desktopCapturer;
const ipcRenderer = require("electron").ipcRenderer;

// Call Electron API to list all available screens
desktopCapturer
  .getSources({ types: ["window", "screen"] })
  .then(async (sources) => {
    const list = document.getElementById("list-of-screens");
    sources.forEach((source) => {
      // Create a new element for each screen and add it to the list of screens
      // ...
    });
  });

// Send the selected screen to the main process
function sendScreenSelection() {
  ipcRenderer.send(
    "screen-share-selected",
    availableScreens[htmlElements.indexOf(selectedElement)].id
  );
  closeWindow();
}

// Cancel screen selection
function cancelSelection() {
  ipcRenderer.send("screen-share-selected", undefined);
  closeWindow();
}

// ...
```

This `modal.html` file utilizes the `desktopCapturer` API to list all available screens and provides functionality for selecting or canceling screen sharing. The `sendScreenSelection()` method is responsible for sending the selected screen to the main process when the user clicks the "_Share Screen_" button.

<div class="grid-container">

<div class="grid-100"><p><a class="glightbox" href="../../../../assets/images/openvidu-electron-screen.png" data-type="image" data-width="100%" data-height="auto" data-desc-position="bottom"><img src="../../../../assets/images/openvidu-electron-screen.png" loading="lazy"/></a></p></div>

</div>

The same is done by the `cancelSelection()` method when the user clicks the "_Cancel_" button but sending `undefined` as the selected screen.

Next, as the **index.js** file is responsible for receiving the selected screen and publishing it to the room, we need to handle the `screen-share-selected` event in the main process:

```javascript
ipcRenderer.on("screen-share-ready", async (event, sourceId) => {
  if (sourceId) {
    try {
      const stream = await navigator.mediaDevices.getUserMedia({
        audio: false,
        video: {
          mandatory: {
            chromeMediaSource: "desktop",
            chromeMediaSourceId: sourceId,
          },
        },
      });

      const track = stream.getVideoTracks()[0];
      const screenPublication = await room.localParticipant.publishTrack(track);
      isScreenShared = true;
      screenSharePublication = screenPublication;
      const element = screenPublication.track.attach();
      element.id = screenPublication.trackSid;
      element.className = "removable";
      document.getElementById("local-participant").appendChild(element);
    } catch (error) {
      console.error("Error enabling screen sharing", error);
    }
  }
});
```

When the screen sharing dialog sends the selected screen to the main process, the `screen-share-ready` event is fired. This event is handled by the `ipcRenderer.on('screen-share-ready')` method, which publishes the selected screen to the room. This code block performs the following steps:

1. Checks if a source ID is provided.
2. Creates a new track using the `navigator.mediaDevices.getUserMedia()` method.
3. Publishes the track to the room using the `room.localParticipant.publishTrack()` method.
4. Attaches the track to the HTML page using `screenPublication.track.attach()`.
5. Updates screen sharing status and displays the screen element on the page.

<h4 markdown>Disabling screen sharing</h4>

When the user clicks the _Toggle screen sharing_ button again, the `toggleScreenShare()` method is called again. This time, the `isScreenShared` variable is set to `false`, and the `stopScreenSharing()` method is called.

```javascript
async function toggleScreenShare() {
  const enabled = !isScreenShared;

  if (enabled) {
    openScreenShareModal();
  } else {
    await stopScreenSharing();
  }
}
```

The `stopScreenSharing()` method is responsible for disabling screen sharing. Here's how it's implemented:

```javascript
async function stopScreenSharing() {
  try {
    await room.localParticipant.unpublishTrack(screenSharePublication.track);
    isScreenShared = false;
    const trackSid = screenSharePublication?.trackSid;

    if (trackSid) {
      document.getElementById(trackSid)?.remove();
    }
    screenSharePublication = undefined;
  } catch (error) {
    console.error("Error stopping screen sharing", error);
  }
}
```

This method unpublishes the screen sharing track from the room using the `room.localParticipant.unpublishTrack()` method. It also removes the screen sharing element from the HTML page and updates the screen sharing status.
