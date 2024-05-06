# openvidu-ionic

[Source code :simple-github:](https://github.com/OpenVidu/openvidu-livekit-tutorials){ .md-button target=\_blank }

This tutorial is a simple Ionic application build with [Capacitor](https://capacitorjs.com/){:target="\_blank"} and [Angular](https://angular.io/){:target="\_blank"}. It is based on the [openvidu-angular](../application-client/angular.md) tutorial, so we recommend you to read it before this one.

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

cd openvidu-livekit-tutorials/openvidu-ionic
npm install
```

You have two options for running the client application: **browser-based** or **mobile device-based**. Choose the option that best fits your needs:

=== ":fontawesome-solid-desktop:{.icon .lg-icon .tab-icon} Browser"

    To run the application in a browser, you'll need to start the Ionic server. To do so, run the following command:

    ```bash
    npm start
    ```

    After the server is up and running, you can test the application by visiting [`http://localhost:8100`](http://localhost:8100){:target="\_blank"}. You should see a screen like this:


    To show the app with a mobile device appearance, open the dev tools in your browser. Find the button to adapt the viewport to a mobile device aspect ratio. You may also choose predefined types of devices to see the behavior of your app in different resolutions.


    <div class="grid-container">

    <div class="grid-50"><p><a class="glightbox" href="../../../../assets/images/ionic-chrome1.png" data-type="image" data-width="100%" data-height="auto" data-desc-position="bottom"><img src="../../../../assets/images/ionic-chrome1.png" loading="lazy"/></a></p></div>

    <div class="grid-50"><p><a class="glightbox" href="../../../../assets/images/ionic-chrome2.png" data-type="image" data-width="100%" data-height="auto" data-desc-position="bottom"><img src="../../../../assets/images/ionic-chrome2.png" loading="lazy"/></a></p></div>

    </div>

=== ":fontawesome-solid-mobile-screen-button:{.icon .lg-icon .tab-icon} Mobile"

    Running the tutorial on a mobile device presents additional challenges compared to running it in a browser, mainly due to the application being launched on a different device, such as an Android smartphone or iPhone, rather than our computer. To overcome these challenges, the following steps need to be taken:

    1. **Localhost limitations:**

    	The usage of `localhost` in our Ionic app is restricted, preventing seamless communication between the application client and the server.

    2. **Serve over local network:**

    	The application must be served over our local network to enable communication between the device and the server.

    3. **Secure connection requirement for WebRTC API:**

    	The WebRTC API demands a secure connection for functionality outside of localhost, necessitating the serving of the application over HTTPS.


    To simplify this process, the tutorial provides a script designed to automatically handle all of the above for you. It sets up a unique HTTPS entry point using [Caddy](https://caddyserver.com/){:target="\_blank"}  to facilitate communication with the server.

    Now, let's explore how to run the application on a mobile device.


    !!! warning "Requirements"
    	1. Before running the application on a mobile device, make sure that the device is connected to the same network as your PC and the mobile is connected to the PC via USB.

    	2. [Docker](https://docs.docker.com/engine/){target="_blank"} is required to launch the proxy server.

    === ":fontawesome-brands-android:{.icon .lg-icon .tab-icon} Android"

    	```bash
    	npm run android
    	```

    === ":fontawesome-brands-apple:{.icon .lg-icon .tab-icon} iOS"

    	You will need [Ruby](https://www.ruby-lang.org/en/documentation/installation/){target="_blank"} and [Cocoapods](https://guides.cocoapods.org/using/getting-started.html){target="_blank"} installed in your computer.

    	The app must be signed with a development team. To do so, open the project in **Xcode** and select a development team in the **Signing & Capabilities** editor.

    	```bash
    	npm run ios
    	```


    This command will get your local IP address, assign it to the `externalIp` variable in the `src/environments/environment.ts` file, and launch the proxy server.

    After that, the script will ask you for the device you want to run the application on. You should select the real device you have connected to your computer.

    Once the mobile device has been selected, the script will launch the application on the device.

    When you run this command, you should see a message like this pop up at some point:

    <div class="grid-container">

    <div class="grid-50"><p><a class="glightbox" href="../../../../assets/images/ionic1.png" data-type="image" data-width="50%" data-height="auto" data-desc-position="bottom"><img src="../../../../assets/images/ionic1.png" loading="lazy"/></a></p></div>

    <div class="grid-50"><p><a class="glightbox" href="../../../../assets/images/ionic3.png" data-type="image" data-width="50%" data-height="auto" data-desc-position="bottom"><img src="../../../../assets/images/ionic3.png" loading="lazy"/></a></p></div>

    </div>

## Understanding the code

As this tutorial is based on the [openvidu-angular](../application-client/angular.md) tutorial but using **Ionic**, we will only explain the new code added to the client application. If you want to understand the rest of the code, please read the [openvidu-angular](../application-client/angular.md) tutorial.

### Preparing for development

The first thing we need to comment on is the `home.page.ts` file. This file contains the logic of the application, and it is where we will add the code to connect to the OpenVidu Server and start the video call.

As mobile device and backend communication may be hard to set up, this class includes the necessary code to run the application in a mobile device.

```typescript
export class HomePage {
	APPLICATION_SERVER_URL = 'http://localhost:5000/';
	WS_LIVEKIT_URL = 'ws://localhost:7880/';
	private IS_DEVICE_DEV_MODE = false;

	// ...

	ngOnInit() {
		/**
		 * WARNING!! To make the mobile development easier, this code allows
		 * using your local IP address for communicating with the backend.
		 * For production uses, the server should be accessible from the Internet
		 * and the code below should be removed.
		 */
		if (this.platform.is('hybrid') && environment.externalIp) {
			this.IS_DEVICE_DEV_MODE = true;
			this.prepareForDeviceDevelopment();
		}
	}

	// ...

	private prepareForDeviceDevelopment() {
		// Pointing to our proxy server through https/wss and our local IP address
		this.APPLICATION_SERVER_URL = `https://${environment.externalIp}:5001/`;
		this.WS_LIVEKIT_URL = `wss://${environment.externalIp}:5001/`;
	}
}
```

This code allows us to use our local IP address for communicating with the backend through the poxy server. This is useful for development purposes and it is only used when the application is running on a mobile device.

Once the development environment is set up, we are ready to ask for the token and connect to the OpenVidu Server. To do so, we will add the following code to the `home.page.ts` file:

```typescript
export class HomePage {
	// ...
	async joinRoom() {
		// ...

		try {
			const token = await this.getToken(
				this.myRoomName,
				this.myParticipantName
			);

			if (!this.IS_DEVICE_DEV_MODE) {
				// Get the Livekit WebSocket URL from the token metadata if not in device dev mode
				this.WS_LIVEKIT_URL = this.getLivekitUrlFromMetadata(token);
			}

			await this.room.connect(livekitUrl, token);

			// ...
		} catch (error) {
			console.log(
				'There was an error connecting to the room:',
				error.code,
				error.message
			);
		}
	}
}
```

This code will ask for the token and connect to the Livekit Server. If the application is not running on a mobile device, the `getLivekitUrlFromMetadata()` method will be called to get the Livekit WebSocket URL from the token metadata.

### Managing media devices

The tutorial provides the functionality to manage the media devices (camera and microphone) from the application. To do so, we will add the following code to the `home.page.ts` file:

#### Managing the camera

```typescript
export class HomePage {
	private cameras: MediaDeviceInfo[] = [];
	private cameraSelected!: MediaDeviceInfo;

	// ...
	async toggleCamera() {
		if (this.room) {
			const enabled = !this.room.localParticipant.isCameraEnabled;
			await this.room.localParticipant.setCameraEnabled(enabled);
			this.refreshVideos();
			this.cameraIcon = enabled ? 'videocam' : 'eye-off';
		}
	}

	async swapCamera() {
		try {
			const newCamera = this.cameras.find(
				(cam) => cam.deviceId !== this.cameraSelected.deviceId
			);

			if (newCamera && this.room) {
				await this.room.switchActiveDevice('videoinput', newCamera.deviceId);
				this.cameraSelected = newCamera;

				this.refreshVideos();
			}
		} catch (error) {
			console.error(error);
		}
	}

	// ...
}
```

#### Managing the microphone

```typescript
export class HomePage {
	// ...

	async toggleMicrophone() {
		if (this.room) {
			const enabled = !this.room.localParticipant.isMicrophoneEnabled;
			await this.room.localParticipant.setMicrophoneEnabled(enabled);
			this.microphoneIcon = enabled ? 'mic' : 'mic-off';
		}
	}
	// ...
}
```

## Specific mobile requirements

The following configuration is required to run the application on an Android and iOS device. Although the **openvidu-ionic tutorial already includes this configuration**, we recommend you to read the following sections to understand all the requirements.

Both platforms install the [@jcesarmobile/ssl-skip](https://github.com/jcesarmobile/ssl-skip#readme) plugin. This plugin allows the application to skip the SSL certificate verification. This is necessary because the proxy server uses a self-signed certificate. This **MUST BE REMOVED** in production environments.

=== ":fontawesome-brands-android:{.icon .lg-icon .tab-icon} Android"

    <h3>Requesting media permissions</h3>

    The application must include media permissions for accessing the device's camera and microphone. These permissions are located in the `android/app/src/main/AndroidManifest.xml` file:

    ```xml
    <uses-permission android:name="android.permission.CAMERA" />
    <uses-permission android:name="android.permission.RECORD_AUDIO" />
    <uses-permission android:name="android.permission.MODIFY_AUDIO_SETTINGS" />
    <uses-permission android:name="android.permission.INTERNET" />
    ```

    With the aim of request these permissions, the application must install the [@awesome-cordova-plugins/android-permissions](https://github.com/danielsogl/awesome-cordova-plugins#readme){:target="\_blank"} plugin. This plugin is already installed in the `package.json` file.

    Using this plugin, the application can request the permissions after connecting to the Room. To do so, we will add the following code to the `home.page.ts` file:

    ```typescript
    export class HomePage {

    	// ...

    	ANDROID_PERMISSIONS = [
    		this.androidPermissions.PERMISSION.CAMERA,
    		this.androidPermissions.PERMISSION.RECORD_AUDIO,
    		this.androidPermissions.PERMISSION.MODIFY_AUDIO_SETTINGS,
    	];

    	// ...

    	private async checkAndroidPermissions(): Promise<void> {
    		await this.platform.ready();
    		try {
    		await this.androidPermissions.requestPermissions(
    			this.ANDROID_PERMISSIONS
    		);
    		const promisesArray: Promise<any>[] = [];
    		this.ANDROID_PERMISSIONS.forEach((permission) => {
    			console.log('Checking ', permission);
    			promisesArray.push(this.androidPermissions.checkPermission(permission));
    		});
    		const responses = await Promise.all(promisesArray);
    		let allHasPermissions = true;
    		responses.forEach((response, i) => {
    			allHasPermissions = response.hasPermission;
    			if (!allHasPermissions) {
    			throw new Error('Permissions denied: ' + this.ANDROID_PERMISSIONS[i]);
    			}
    		});
    		} catch (error) {
    		console.error('Error requesting or checking permissions: ', error);
    		throw error;
    		}
    	}

    	// ...
    }
    ```

    <h3>Rendering videos</h3>

    We noticed that the application was not working properly on Android devices (at least on our test devices). The video elements were not being displayed correctly in certain situations.

    We added a workaround to fix this issue. The workaround is located in the `home.page.ts` file and it is called `refreshVideos()` and consists of the following code:

    ```typescript
    private refreshVideos() {
    	if (this.platform.is('hybrid') && this.platform.is('android')) {
    		// Workaround for Android devices
    		setTimeout(() => {
    			const refreshedElement = document.getElementById(
    			'refreshed-workaround'
    			);
    			if (refreshedElement) {
    			refreshedElement.remove();
    			} else {
    			const p = document.createElement('p');
    			p.id = 'refreshed-workaround';
    			document.getElementById('room')?.appendChild(p);
    			}
    		}, 250);
    	}
    }
    ```

    It is called every time the camera track is regenerated and basically update the DOM adding or romoving a HTML element (a paragraph element in this case). This forces the browser to refresh the video elements and display them correctly.

=== ":fontawesome-brands-apple:{.icon .lg-icon .tab-icon} iOS"

    The application must include the following configuration files with the aim of allowing the media access to the device's camera and microphone. These config are located in the `src/ios/App/App/Info.plist` file:

    ```xml
    <key>NSCameraUsageDescription</key>
    <string>This Application uses your camera to make video calls.</string>
    <key>NSMicrophoneUsageDescription</key>
    <string>This Application uses your microphone to make calls.</string>
    ```
