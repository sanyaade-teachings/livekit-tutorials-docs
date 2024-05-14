# openvidu-recording

[Source code :simple-github:](https://github.com/OpenVidu/openvidu-livekit-tutorials){ .md-button target=\_blank }

This OpenVidu application is a simple videoconference app that allows you to record the room. It includes a basic frontend based on [openvidu-js](../application-client/javascript.md) and two different server implementations: one using **Node.js**, which you can learn more about in the [openvidu-basic-node tutorial](../application-server/node.md), and the other using **Java**, covered in the [openvidu-basic-java tutorial](../application-server/java.md).

## Running this tutorial

Running this tutorial is straightforward, and here's what you'll need:

### 1. OpenVidu Server Installation

--8<-- "docs/tutorials/shared/run-livekit-server.md"

!!! info "Livekit Egress"
If you want to use the Livekit distribution instead of OpenVidu, you will need to configure the egress server. You can find more information about this in the [Livekit documentation](https://docs.livekit.io/egress-ingress/egress/self-hosting/){:target="\_blank"}.

<h3>2. Launch the client and server application</h3>

Both application servers contain the same client functionality, so you can choose the one you prefer.

=== ":fontawesome-brands-node-js:{.icon .lg-icon .tab-icon} Node"

    To run this server application, you'll need [NPM](https://docs.npmjs.com/downloading-and-installing-node-js-and-npm){:target="\_blank"} installed on your development computer.

    1. Clone the repository
    ```bash
    git clone https://github.com/OpenVidu/openvidu-livekit-tutorials.git
    ```
    2. Install dependencies
    ```bash
    cd openvidu-livekit-tutorials/openvidu-recording-node
    npm install
    ```

    3. Run the tutorial
    ```bash
    npm run start
    ```

=== ":fontawesome-brands-java:{.icon .lg-icon .tab-icon} Java"

    To run this server application, you'll need [Java](https://www.java.com/en/download/manual.jsp){:target="\_blank"} and [Maven](https://maven.apache.org/){:target="\_blank"} installed on your development computer.


    1. Clone the repository
    ```bash
    git clone https://github.com/OpenVidu/openvidu-livekit-tutorials.git
    ```
    2. Run the tutorial
    ```bash
    cd openvidu-livekit-tutorials/openvidu-recording-java
    mvn package exec:java
    ```

The application will be available at [https://localhost:5000](https://localhost:5000){:target="\_blank"}.

<div class="grid-container">

<div class="grid-50"><p><a class="glightbox" href="../../../../assets/images/recording-1.png" data-type="image" data-width="100%" data-height="auto" data-desc-position="bottom"><img src="../../../../assets/images/recording-1.png" loading="lazy"/></a></p></div>

<div class="grid-50"><p><a class="glightbox" href="../../../../assets/images/recording-2.png" data-type="image" data-width="100%" data-height="auto" data-desc-position="bottom"><img src="../../../../assets/images/recording-2.png" loading="lazy"/></a></p></div>

</div>

## Understanding the code

<h3>Frontend Enhancements</h3>

The client application, which has built upon the fundation of the [openvidu-js tutorial](../application-client/javascript.md), has been extended with additional functionalities, introducing new buttons to facilitate actions such as starting and stopping recordings, as well as listing and deleting recorded rooms. These enhancements empower users with more control over the video room.

When these newly introduced buttons are interacted with, the client triggers requests to the REST API endpoints of the server application. These API calls enable seamless communication between the frontend and backend, facilitating the desired actions.

<h3>Backend Enhancements</h3>

In the next sections, we'll dive into detailed explanations of specific REST endpoints related to recording features. These improvements were inspired by the [openvidu-basic-node tutorial](../application-server/node.md) and [openvidu-basic-java tutorial](../application-server/java.md) code examples.

=== ":fontawesome-brands-node-js:{.icon .lg-icon .tab-icon} Node"

    The **Node.js** project includes the following essential files:

    - `server.js`: This file holds the server application.

    And the following essential **frontend** files under the `/public` directory:

    - `index.html`: Inside here, you'll find the client application's main HTML file.
    - `app.js`: This file houses the client application's logic and functionality.
    - `style.css`: This file contains the client application's styling.

=== ":fontawesome-brands-java:{.icon .lg-icon .tab-icon} Java"

    The **Spring Boot** project includes the following essential **backend** files under the `src/main/java` directory:

    - `App.java`: This file holds the server application.
    - `MyRestController.java`: This file is the REST controller for the token and recordings endpoints.

    And the following essential **frontend** files under the `src/main/resources/static` directory:

    - `index.html`: Inside here, you'll find the client application's main HTML file.
    - `app.js`: This file houses the client application's logic and functionality.
    - `style.css`: This file contains the client application's styling.

Let's take a closer look at the code.

### Adding recording

For handling recording features, this tutorial has been enhanced with new REST API endpoints. These endpoints provide the following functionality:

- **`POST /recordings/start`** - This endpoint starts the recording of a room.
- **`POST /recordings/stop`** - This endpoint stops the recording of a room.
- **`GET /recordings/list`** - This endpoint lists all the recordings stored in the `/recordings` folder.
- **`DELETE /recordings`** - This endpoint deletes all the recordings stored in the `/recordings` folder.

<h4 markdown>Start recording</h4>

There are two ways to start recording a room:

- `COMPOSED`: The server will compose all the published tracks into a single video file.

- `INDIVIDUAL`: The server will record each published track into a separate video file.

!!! warning TODO
Update this section properly when openvidu-dev is stable

All recordings will be stored in the `/recordings` folder by default.

=== ":fontawesome-brands-node-js:{.icon .lg-icon .tab-icon} Node"

    ```javascript hl_lines="26-34 36-43"
    const egressClient = new EgressClient(
    	livekitUrlHostname,
    	LIVEKIT_API_KEY,
    	LIVEKIT_API_SECRET
    );

    app.post('/recordings/start', async function (req, res) {
    	const {
    		roomName,
    		outputMode,
    		videoOnly,
    		audioOnly,
    		audioTrackId,
    		videoTrackId,
    	} = req.body;

    	const output = {
    		fileType: 0, // file type chosen based on codecs
    		filepath: `/recordings/${roomName}-${new Date().getTime()}`,
    		disableManifest: true,
    	};

    	try {
    		let egressInfo;
    		if (outputMode === 'COMPOSED') {
    			egressInfo = await egressClient.startRoomCompositeEgress(
    				roomName,
    				output,
    				{
    					layout: 'grid',
    					audioOnly,
    					videoOnly,
    				}
    			);
    		} else if (outputMode === 'INDIVIDUAL') {
    			egressInfo = await egressClient.startTrackCompositeEgress(
    				roomName,
    				output,
    				{
    					audioTrackId,
    					videoTrackId,
    				}
    			);
    		} else {
    			res.status(400).json({ message: 'outputMode is required' });
    			return;
    		}
    		res.status(200).json({ message: 'recording started', info: egressInfo });
    	} catch (error) {
    		console.log('Error starting recording', error);
    		res.status(200).json({ message: 'error starting recording' });
    	}
    });
    ```

=== ":fontawesome-brands-java:{.icon .lg-icon .tab-icon} Java"

    ```java hl_lines="23-25 27-29"

    @RequestMapping(value = "/recordings/start", method = RequestMethod.POST)
    public ResponseEntity<?> startRecording(@RequestBody Map<String, Object> params) {
    	try {
    		String roomName = (String) params.get("roomName");
    		String outputMode = (String) params.get("outputMode");
    		Boolean videoOnly = (Boolean) params.get("videoOnly");
    		Boolean audioOnly = (Boolean) params.get("audioOnly");
    		String audioTrackId = (String) params.get("audioTrackId");
    		String videoTrackId = (String) params.get("videoTrackId");

    		Builder outputBuilder = LivekitEgress.EncodedFileOutput.newBuilder()
    				.setFileType(EncodedFileType.DEFAULT_FILETYPE)
    				.setFilepath("/recordings/" + roomName + "-" + new Date().getTime())
    				.setDisableManifest(true);

    		EncodedFileOutput output = outputBuilder.build();

    		System.out.println("Starting recording " + roomName);

    		LivekitEgress.EgressInfo egressInfo;

    		if ("COMPOSED".equals(outputMode)) {
    			egressInfo = this.egressClient
    					.startRoomCompositeEgress(roomName, output, "grid", null, null, audioOnly, videoOnly)
    					.execute().body();
    		} else if ("INDIVIDUAL".equals(outputMode)) {
    			System.out.println("Starting INDIVIDUAL recording " + roomName);
    			egressInfo = this.egressClient.startTrackCompositeEgress(roomName, output, audioTrackId, videoTrackId)
    					.execute().body();
    		} else {
    			return ResponseEntity.badRequest().body("outputMode is required");
    		}

    		return ResponseEntity.ok().body(generateEgressInfoResponse(egressInfo));

    	} catch (Exception e) {
    		System.out.println("Error starting recording " + e.getMessage());
    		return ResponseEntity.badRequest().body("Error starting recording");
    	}
    }

    ```

<h4 markdown>Stop recording</h4>

Once the recording has started, the user can stop it by clicking the `Stop recording` button. This button will trigger a request to the `/recordings/stop` endpoint.
This endpoint will stop the recording and return the recording information.

=== ":fontawesome-brands-node-js:{.icon .lg-icon .tab-icon} Node"

    ```javascript hl_lines="10"
    app.post('/recordings/stop', async function (req, res) {
    	const recordingId = req.body.recordingId;
    	try {
    		if (!recordingId) {
    			res.status(400).json({ message: 'recordingId is required' });
    			return;
    		}

    		console.log(`Stopping recording ${recordingId}`);
    		const egressInfo = await egressClient.stopEgress(recordingId);
    		res.status(200).json({ message: 'recording stopped', info: egressInfo });
    	} catch (error) {
    		console.log('Error stopping recording', error);
    		res.status(200).json({ message: 'error stopping recording' });
    	}
    });
    ```

=== ":fontawesome-brands-java:{.icon .lg-icon .tab-icon} Java"

    ```java hl_lines="11"
    @RequestMapping(value = "/recordings/stop", method = RequestMethod.POST)
    public ResponseEntity<?> stopRecording(@RequestBody Map<String, Object> params) {

    	String recordingId = (String) params.get("recordingId");

    	if (recordingId == null) {
    		return ResponseEntity.badRequest().body("recordingId is required");
    	}

    	try {
    		LivekitEgress.EgressInfo egressInfo = this.egressClient.stopEgress(recordingId).execute().body();
    		return ResponseEntity.ok().body(generateEgressInfoResponse(egressInfo));
    	} catch (Exception e) {
    		System.out.println("Error stoping recording " + e.getMessage());
    		return ResponseEntity.badRequest().body("Error stoping recording");
    	}
    }
    ```

<h4 markdown>Delete recordings</h4>

This tutorial also provides the ability to delete all recordings stored in the `/recordings` folder. This functionality is available through the `Delete recordings` button, which triggers a request to the `/recordings` endpoint.

=== ":fontawesome-brands-node-js:{.icon .lg-icon .tab-icon} Node"

    As the recordings are stored in local files, we use the `fs` module to delete them. The `fs` module is a Node.js core module that provides an API for interacting with the file system.

    ```javascript
    app.delete('/recordings', function (req, res) {
    	fs.readdirSync(RECORDINGS_PATH, { recursive: true }).forEach((value) => {
    		fs.unlinkSync(`${RECORDINGS_PATH}/${value}`);
    		if (fs.existsSync(`public/${value}`)) {
    			fs.unlinkSync(`public/${value}`);
    		}
    	});
    	res.status(200).json({ message: 'All recordings deleted' });
    });

    ```

=== ":fontawesome-brands-java:{.icon .lg-icon .tab-icon} Java"

    ```java
    @RequestMapping(value = "/recordings", method = RequestMethod.DELETE)
    public ResponseEntity<?> deleteRecordings() {

    	try {
    		File recordingsDir = ResourceUtils.getFile("classpath:static");
    		deleteFiles(new File(RECORDINGS_PATH));
    		deleteFiles(new File(recordingsDir.getAbsolutePath()));
    		JSONObject response = new JSONObject();
    		response.put("message", "All recordings deleted");

    		return ResponseEntity.ok().body(response.toMap());
    	} catch (IOException e) {
    		e.printStackTrace();
    		return ResponseEntity.status(HttpStatus.INTERNAL_SERVER_ERROR).body("Error deleting recordings");
    	}
    }
    ```

<h4 markdown>List recordings</h4>

The `List recordings` button triggers a request to the `/recordings/list` endpoint. This endpoint will return the list of recordings stored in the `/recordings` folder.

=== ":fontawesome-brands-node-js:{.icon .lg-icon .tab-icon} Node"

    ```javascript
    app.get('/recordings/list', function (req, res) {
    	const recordings = [];
    	fs.readdirSync(RECORDINGS_PATH, { recursive: true }).forEach((value) => {
    		// copy file to public folder for development purposes
    		fs.copyFileSync(`${RECORDINGS_PATH}/${value}`, `public/${value}`);
    		const newRec = { name: value, path: `/${value}` };
    		recordings.push(newRec);
    	});
    	res.status(200).json({ recordings });
    });
    ```

=== ":fontawesome-brands-java:{.icon .lg-icon .tab-icon} Java"

    ```java
    @RequestMapping(value = "/recordings/list", method = RequestMethod.GET)
    public ResponseEntity<?> listRecordings() {

    	List<JSONObject> recordings = new ArrayList<>();

    	try {
    		File recordingsDir = ResourceUtils.getFile("classpath:static");
    		Files.walk(Path.of(RECORDINGS_PATH)).forEach(filePath -> {
    			JSONObject recordingsMap = new JSONObject();

    			if (Files.isRegularFile(filePath)) {
    				String fileName = filePath.getFileName().toString();
    				String destinationPath = recordingsDir.getAbsolutePath() + File.separator + fileName;

    				try {
    					Files.copy(filePath, Path.of(destinationPath), StandardCopyOption.REPLACE_EXISTING);
    				} catch (IOException e) {
    					e.printStackTrace();
    				}

    				recordingsMap.put("name", fileName);
    				recordingsMap.put("path", "/" + fileName);
    				recordings.add(recordingsMap);
    			}
    		});
    	} catch (IOException e) {
    		e.printStackTrace();
    		return new ResponseEntity<>(HttpStatus.INTERNAL_SERVER_ERROR);
    	}

    	JSONObject response = new JSONObject();
    	response.put("recordings", recordings);
    	return new ResponseEntity<>(response.toMap(), HttpStatus.OK);
    }
    ```

## Deploying openvidu-recording (TODO)

=== ":fontawesome-brands-node-js:{.icon .lg-icon .tab-icon} Node"

    <h3> 1. Build the docker image</h3>

    Under the root project folder, you can see the `openvidu-recording-node/docker/` directory. Here it is included all the required files yo make it possible the deployment with OpenVidu.

    First of all, you will need to create the **openvidu-recording-node** docker image. Under `openvidu-recording-node/docker/` directory you will find the `create_image.sh` script. This script will create the docker image with the openvidu-recording-node server and the static files.


    ```bash
    ./create_image.sh openvidu/openvidu-recording-node-demo:X.Y.Z
    ```

    This script will create an image named `openvidu/openvidu-recording-node-demo:X.Y.Z`. This name will be used in the next step.

=== ":fontawesome-brands-java:{.icon .lg-icon .tab-icon} Java"

    <h3> 1. Build the docker image</h3>

    Under the root project folder, you can see the `openvidu-recording-java/docker/` directory. Here it is included all the required files yo make it possible the deployment with OpenVidu.

    First of all, you will need to create the **openvidu-recording-java** docker image. Under `openvidu-recording-java/docker/` directory you will find the `create_image.sh` script. This script will create the docker image with the openvidu-recording-java server and the static files.


    ```bash
    ./create_image.sh openvidu/openvidu-recording-java-demo:X.Y.Z
    ```

    This script will create an image named `openvidu/openvidu-recording-java-demo:X.Y.Z`. This name will be used in the next step.

<h3> 2. Deploy the docker image </h3>

Time to deploy the docker image. You can follow the [Deploy OpenVidu based application with Docker](/deployment/deploying-openvidu-apps/#with-docker) guide for doing this.
