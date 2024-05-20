# openvidu-js-screen-share

[Source code :simple-github:](https://github.com/OpenVidu/openvidu-tutorials){ .md-button target=\_blank }

This tutorial shows how to use screen sharing in Livekit applications. It is based on the [openvidu-js](../application-client/javascript.md) tutorial, so we recommend you to read it before this one.

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

To run the client application tutorial, you'll need a HTTP web server installed on your development computer. If you have Node.js installed, you can easily set up [http-server](https://github.com/indexzero/http-server){:target="\_blank"}. Here's how to install it:

```bash
npm install --location=global http-server
```

After installing http-server, serve the tutorial by executing the following command:

```bash
# Using the same repository openvidu-livekit-tutorials from step 2
http-server openvidu-tutorials/openvidu-js-screen-share/web
```

Once the server is up and running, you can test the application by visiting [`http://localhost:8080`](http://localhost:8080){:target="\_blank"}. You should see a screen like this:

<div class="grid-container">

<div class="grid-50"><p><a class="glightbox" href="../../../../assets/images/insecure-join.png" data-type="image" data-width="100%" data-height="auto" data-desc-position="bottom"><img src="../../../../assets/images/insecure-join.png" loading="lazy"/></a></p></div>

<div class="grid-50"><p><a class="glightbox" href="../../../../assets/images/insecure-session.png" data-type="image" data-width="100%" data-height="auto" data-desc-position="bottom"><img src="../../../../assets/images/insecure-session.png" loading="lazy"/></a></p></div>

</div>

!!! info "Test with other devices"

    To test the application with other devices in your network, visit this **[FAQ](troubleshooting/#3-test-applications-in-my-network-with-multiple-devices)**

## Understanding the code

As this tutorial is based on the [openvidu-js](../application-client/javascript.md) tutorial but adding screen sharing, we will only explain the new code added to the client application. If you want to understand the rest of the code, please read the [openvidu-js](../application-client/javascript.md) tutorial.

### Adding the HTML button

The first thing we need to do is to add a button to the client application to start the screen sharing. To do so, we will add the following code to the `index.html` file:

```html
<input
  class="btn btn-large"
  type="button"
  id="buttonScreenShare"
  onmouseup="toggleScreenShare()"
  value="Screen share"
/>
```

This button will call the `toggleScreenShare()` function when clicked. This function will be responsible for starting and stopping the screen sharing.

### Adding the logic

Now we need to add the logic to start and stop the screen sharing. To do so, we will add the following code to the `index.js` file:

```javascript
var isScreenShared = false;
var screenSharePublication;

// ...

async function toggleScreenShare() {
  console.log("Toggling screen share");
  const enabled = !isScreenShared;

  if (enabled) {
    // Enable screen sharing
    // ...
  } else {
    // Disable screen sharing
    await stopScreenSharing();
  }
}
```

#### Enabling screen sharing

As you can see, we have added a variable called `isScreenShared` to keep track of the screen sharing status. We have also added a variable called `screenSharePublication` to keep track of the screen sharing publication.

Now we need to implement the logic when the screen sharing is enabled. To do so, we will add the following code to the `toggleScreenShare()` function:

```javascript
async function toggleScreenShare() {
  // ...

  if (enabled) {
    // Enable screen sharing
    try {
      screenSharePublication =
        await room.localParticipant?.setScreenShareEnabled(enabled);
    } catch (error) {
      console.error("Error enabling screen sharing", error);
    }

    if (screenSharePublication) {
      console.log("Screen sharing enabled", screenSharePublication);
      isScreenShared = enabled;

      // Attach the screen share track to the video container
      const element = screenSharePublication.track.attach();
      element.id = screenSharePublication.trackSid;
      element.className = "removable";
      document.getElementById("video-container").appendChild(element);

      // Add user data for the screen share
      appendUserData(element, `${myUserName}_SCREEN`);

      // Listen for the 'ended' event to handle screen sharing stop
      screenSharePublication.addListener("ended", async () => {
        console.debug("Clicked native stop button. Stopping screen sharing");
        await stopScreenSharing();
      });
    }
  } else {
    // Disable screen sharing
    await stopScreenSharing();
  }
}
```

As you can see, we are calling the `setScreenShareEnabled()` method of the local participant to enable the screen sharing. This method will return a `Publication` object if the screen sharing was enabled successfully.

After the screenshare publication has been returned successfully, we will update the `isScreenShared` variable to `true` and attach the screen share track to the video container. We will also add the user data to the screen share track to differentiate it from the camera track.

Finally, we will listen for the `ended` event of the screen share publication to handle the screen sharing stop through the native browser stop button.

#### Disabling screen sharing

Now we need to implement the logic when the screen sharing is disabled. To do so, we will add the following code to the `toggleScreenShare()` function:

```javascript
async function stopScreenSharing() {
  try {
    await room.localParticipant?.setScreenShareEnabled(false);
    isScreenShared = false;
    const trackSid = screenSharePublication?.trackSid;

    if (trackSid) {
      document.getElementById(trackSid)?.remove();
      removeUserData({ identity: `${myUserName}_SCREEN` });
    }
    screenSharePublication = undefined;
  } catch (error) {
    console.error("Error stopping screen sharing", error);
  }
}
```

Here, we also have to call the `setScreenShareEnabled()` method of the local participant to disable the screen sharing with the `false` parameter. Once the screen sharing has been disabled, we will update the `isScreenShared` variable to `false` and remove the screen share track from the video container. We will also remove the user data from the screen share track.

## Deploying openvidu-js-screen-share (TODO)

> This tutorial image is **officially released for OpenVidu** under the name `openvidu/openvidu-js-screen-share-demo:X.Y.Z` so you do not need to build it by yourself. However, if you want to deploy a custom version of openvidu-js-screen-share, you will need to build a new one. You can keep reading for more information.

<h3> 1. Build the docker image</h3>

Under the root project folder, you can see the `openvidu-js-screen-share/docker/` directory. Here it is included all the required files yo make it possible the deployment with OpenVidu.

First of all, you will need to create the **openvidu-js-screen-share** docker image. Under `openvidu-js-screen-share/docker/` directory you will find the `create_image.sh` script. This script will create the docker image with the [openvidu-basic-node](../application-server/node.md) as application server and the static files.

```bash
./create_image.sh openvidu/openvidu-js-screen-share-demo:X.Y.Z
```

This script will create an image named `openvidu/openvidu-js-screen-share-demo:X.Y.Z`. This name will be used in the next step.

<h3> 2. Deploy the docker image </h3>

Time to deploy the docker image. You can follow the [Deploy OpenVidu based application with Docker](#) guide for doing this.
