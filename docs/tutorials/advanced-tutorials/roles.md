# openvidu-roles

[Source code :simple-github:](https://github.com/OpenVidu/openvidu-livekit-tutorials){ .md-button target=\_blank }

This OpenVidu application is a simple videoconference app that allows you to assign roles to participants. It includes a basic frontend based on [openvidu-js](../application-client/javascript.md) and two different server implementations: one using **Node.js**, which you can learn more about in the [openvidu-basic-node tutorial](../application-server/node.md), and the other using **Java**, covered in the [openvidu-basic-java tutorial](../application-server/java.md).

## Running this tutorial

Running this tutorial is straightforward, and here's what you'll need:

### 1. OpenVidu Server Installation

--8<-- "docs/tutorials/shared/run-livekit-server.md"

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
    cd openvidu-livekit-tutorials/openvidu-roles-node
    npm install
    ```

    3. Run the server
    ```bash
    node server.js
    ```

=== ":fontawesome-brands-java:{.icon .lg-icon .tab-icon} Java"

    To run this server application, you'll need [Java](https://www.java.com/en/download/manual.jsp){:target="\_blank"} and [Maven](https://maven.apache.org/){:target="\_blank"} installed on your development computer.


    1. Clone the repository
    ```bash
    git clone https://github.com/OpenVidu/openvidu-livekit-tutorials.git
    ```
    2. Run the server
    ```bash
    cd openvidu-livekit-tutorials/openvidu-roles-java
    mvn package exec:java
    ```

The application will be available at [https://localhost:5000](https://localhost:5000){:target="\_blank"}. You can open it in normal and incognito mode to test the different roles.

<div class="grid-container">

<div class="grid-50"><p><a class="glightbox" href="../../../../assets/images/secure-login.png" data-type="image" data-width="100%" data-height="auto" data-desc-position="bottom"><img src="../../../../assets/images/secure-login.png" loading="lazy"/></a></p></div>

<div class="grid-50"><p><a class="glightbox" href="../../../../assets/images/secure-join.png" data-type="image" data-width="100%" data-height="auto" data-desc-position="bottom"><img src="../../../../assets/images/secure-join.png" loading="lazy"/></a></p></div>

</div>

<div class="grid-container">

<div class="grid-50"><p><a class="glightbox" href="../../../../assets/images/secure-session-1.png" data-type="image" data-width="100%" data-height="auto" data-desc-position="bottom"><img src="../../../../assets/images/secure-session-1.png" loading="lazy"/></a></p></div>

<div class="grid-50"><p><a class="glightbox" href="../../../../assets/images/secure-session-2.png" data-type="image" data-width="100%" data-height="auto" data-desc-position="bottom"><img src="../../../../assets/images/secure-session-2.png" loading="lazy"/></a></p></div>

</div>

!!! info "Test with other devices"

    To test the application with other devices in your network, visit this **[FAQ](troubleshooting/#3-test-applications-in-my-network-with-multiple-devices)**

## Understanding the code

On the frontend side, our tutorial is built upon the foundation of the [openvidu-js tutorial](../application-client/javascript.md). You can refer to that tutorial for a comprehensive understanding of the client application's codebase.

We'll focus on explaining the specific code additions related to roles feature.

=== ":fontawesome-brands-node-js:{.icon .lg-icon .tab-icon} Node"

    To gain a deeper understanding of the existing code within the server application, we highly recommend exploring the [openvidu-basic-node tutorial](../application-server/node.md) on the server side. This resource will provide valuable insights into the pre-existing codebase.

    The **Node.js** project includes the following essential files:

    - `server.js`: This file holds the server application.
    - `public/index.html`: Inside here, you'll find the client application's main HTML file.
    - `public/app.js`: This file houses the client application's logic and functionality.
    - `public/style.css`: This file contains the client application's styling.

=== ":fontawesome-brands-java:{.icon .lg-icon .tab-icon} Java"

    To gain a deeper understanding of the existing code within the server application, we highly recommend exploring the [openvidu-basic-java tutorial](../application-server/java.md) on the server side. This resource will provide valuable insights into the pre-existing codebase.

    The **Spring Boot** project includes the following essential **backend** files under the `src/main/java` directory:

    - `App.java`: This file holds the server application.
    - `LoginController.java`: This file is the REST controller for the login and logout endpoints.
    - `RoomController.java`: This file is the REST controller for the token endpoint.

    And the following essential **frontend** files under the `src/main/resources/static` directory:

    - `index.html`: Inside here, you'll find the client application's main HTML file.
    - `app.js`: This file houses the client application's logic and functionality.
    - `style.css`: This file contains the client application's styling.

Let's take a closer look at the code.

### Authentication

In this tutorial, we've introduced the following new features to enhance the base version:

- Login / Logout
- Participant Roles

To enable these features, two new endpoints have been integrated into the server application REST API:

- `/login`: This endpoint facilitates user login.
- `/logout`: This endpoint enables users to log out from the application.

Additionally, the `/token` endpoint has been modified to incorporate participant roles into the token.

=== ":fontawesome-brands-node-js:{.icon .lg-icon .tab-icon} Node"

    First, we'll store the users' credentials in the `server.js` file for simplicity. However, in a real-world application, you'd want to store this information in a database.

    ```javascript
    const users = [
    	{
    		user: 'publisher1',
    		pass: 'pass',
    		role: 'PUBLISHER',
    	},
    	{
    		user: 'publisher2',
    		pass: 'pass',
    		role: 'PUBLISHER',
    	},
    	{
    		user: 'subscriber',
    		pass: 'pass',
    		role: 'SUBSCRIBER',
    	},
    ];

    // ...
    ```

    Next, we'll add the logic to the `/login` endpoint. This endpoint will receive the user's credentials and validate them against the stored credentials. If the credentials are valid, the endpoint will allow the user to log in and request a token. Otherwise, the endpoint will return an error.

    ```javascript
    // ...
    app.post('/login', (req, res) => {
    	const { user, pass } = req.body;

    	if (login(user, pass)) {
    		// Successful login
    		// Validate session and return OK
    		// Value stored in req.session allows us to identify the user in future requests
    		console.log(`Successful login for user '${user}'`);
    		req.session.loggedUser = user;
    		res.status(200).json({});
    	} else {
    		// Credentials are NOT valid
    		// Invalidate session and return error
    		console.log(`Invalid credentials for user '${user}'`);
    		req.session.destroy();
    		res.status(401).json({ message: 'Invalid credentials' });
    	}
    });

    //...
    ```

    Finally, we'll add the logic to the `/logout` endpoint. This endpoint will invalidate the user's session and log them out of the application.

    ```javascript
    app.post('/logout', (req, res) => {
    	console.log(`'${req.session.loggedUser}' has logged out`);
    	req.session.destroy();
    	res.status(200).json({});
    });
    ```

=== ":fontawesome-brands-java:{.icon .lg-icon .tab-icon} Java"

    First, we'll store the users' credentials in the `LoginController.java` file for simplicity. However, in a real-world application, you'd want to store this information in a database.

    ```java
    @RestController
    public class LoginController {

    	public class MyUser {

    		String name;
    		String pass;
    		String role;

    		public MyUser(String name, String pass, String role) {
    			this.name = name;
    			this.pass = pass;
    			this.role = role;
    		}
    	}

    	public static Map<String, MyUser> users = new ConcurrentHashMap<>();

    	public LoginController() {
    		users.put("publisher1", new MyUser("publisher1", "pass", "PUBLISHER"));
    		users.put("publisher2", new MyUser("publisher2", "pass", "PUBLISHER"));
    		users.put("subscriber", new MyUser("subscriber", "pass", "SUBSCRIBER"));
    	}

    	// ...
    }
    ```

    Next, we'll add the logic to the `/login` endpoint. This endpoint will receive the user's credentials and validate them against the stored credentials. If the credentials are valid, the endpoint will allow the user to log in and request a token. Otherwise, the endpoint will return an error.

    ```java
    @RestController
    public class LoginController {

    	// ...

    	@PostMapping("/login")
    	public ResponseEntity<?> login(@RequestBody(required = true) Map<String, String> params, HttpSession httpSession) {

    		String user = params.get("user");
    		String pass = params.get("pass");
    		Map<String, String> response = new HashMap<String, String>();

    		if (login(user, pass)) {
    			// Successful login
    			// Validate session and return OK
    			// Value stored in req.session allows us to identify the user in future requests
    			httpSession.setAttribute("loggedUser", user);
    			return new ResponseEntity<>(new HashMap<String, String>(), HttpStatus.OK);
    		} else {
    			// Credentials are NOT valid
    			// Invalidate session and return error
    			httpSession.invalidate();
    			response.put("message", "Invalid credentials");
    			return new ResponseEntity<>(response, HttpStatus.UNAUTHORIZED);
    		}
    	}

    	private boolean login(String user, String pass) {
    		if(user.isEmpty() || pass.isEmpty()) return false;
    		return (users.containsKey(user) && users.get(user).pass.equals(pass));
    	}

    	// ...
    }
    ```

    Finally, we'll add the logic to the `/logout` endpoint. This endpoint will invalidate the user's session and log them out of the application.

    ```java
    @RestController
    public class LoginController {

    	// ...

    	@PostMapping("/logout")
    	public ResponseEntity<Void> logout(HttpSession session) {
    		System.out.println("'" + session.getAttribute("loggedUser") + "' has logged out");
    		session.invalidate();
    		return new ResponseEntity<>(HttpStatus.OK);
    	}

    	// ...
    }
    ```

After adding the authentication logic, we'll add the login and logout functionality to the client application.

<div class="grid-container">

<div class="grid-50"><p><a class="glightbox" href="../../../../assets/images/secure-login.png" data-type="image" data-width="100%" data-height="auto" data-desc-position="bottom"><img src="../../../../assets/images/secure-login.png" loading="lazy"/></a></p></div>

<div class="grid-50"><p><a class="glightbox" href="../../../../assets/images/secure-join.png" data-type="image" data-width="100%" data-height="auto" data-desc-position="bottom"><img src="../../../../assets/images/secure-join.png" loading="lazy"/></a></p></div>

</div>

The tutorial includes a simple form to log in of the application. The form is displayed when the user is not logged in. Once the user has logged in, the form is hidden and the user can join a session.

When a user click on the `Login` button, the client application will call the `/login` endpoint explained above. If the login is successful, the client application will be able to join a session and request a token to the `/token` endpoint.

Let's see how the `/token` endpoint has been modified to include the user role.

### Adding roles

Once the user has logged in, the client application will request a token when the user joins a session. To do so, the client application will call the `/token` endpoint. This endpoint will validate the user's session and return a token with the user role.

=== ":fontawesome-brands-node-js:{.icon .lg-icon .tab-icon} Node"

    ```javascript hl_lines="6-10 17-19 23 28 29"
    // ...
    app.post('/token', (req, res) => {

    	const {roomName, participantName} = req.body;

    	if (!isLogged(req.session)) {
    		req.session.destroy();
    		res.status(401).json({ message: 'User not logged' });
    		return;
    	}

    	if (!roomName || !participantName) {
    		res.status(400).json({ message: 'roomName and participantName are required' });
    		return;
    	}

    	const user = users.find((u) => u.user === req.session.loggedUser);
    	const {role, user: nickname} = user;
    	const canPublish = role === 'PUBLISHER';
    	const at = new AccessToken(LIVEKIT_API_KEY, LIVEKIT_API_SECRET, {
    		identity: participantName,
    		// add metadata to the token, which will be available in the participant's metadata
    		metadata: JSON.stringify({ livekitUrl: LIVEKIT_URL, nickname, role }),
    	});
    	at.addGrant({
    		roomJoin: true,
    		room: roomName,
    		canPublish,
    		canSubscribe: true,
    	});
    	res.status(200).json({ token: at.toJwt() });
    });
    ```

    The `/token` endpoint has been enhanced with the following logic:


    - **Validate the user session**: The endpoint checks if the user is logged in; if not, it returns an error.
    - **Getting user role**: It fetches the user role based on their credentials.
    - **Managing permissions**: User permissions are determined based on their role. For example, only publisher users can publish their tracks to the room due to the `canPublish` property.
    - **Adding metadata to the token**: The user role is included in the token's metadata for reference.

=== ":fontawesome-brands-java:{.icon .lg-icon .tab-icon} Java"

    The `RoomController.java` file contains the `/token` endpoint. This endpoint will validate the user's session and return a token with the user role.

    ```java hl_lines="11-14 21-22 32 33 36"
    @RestController
    public class RoomController {

    	@PostMapping("/token")
    	public ResponseEntity<Map<String, String>> getToken(@RequestBody(required = true) Map<String, String> params, HttpSession httpSession) {

    		String roomName = params.get("roomName");
    		String participantName = params.get("participantName");
    		Map<String, String> response = new HashMap<String, String>();

    		if(!isUserLogged(httpSession)) {
    			response.put("message", "User is not logged");
    			return new ResponseEntity<>(response, HttpStatus.UNAUTHORIZED);
    		}

    		if(roomName == null || participantName == null) {
    			response.put("message", "roomName and participantName are required");
    			return new ResponseEntity<>(response, HttpStatus.BAD_REQUEST);
    		}

    		MyUser user = LoginController.users.get(httpSession.getAttribute("loggedUser"));
    		Boolean canPublish = user.role.equals("PUBLISHER");

    		// By default, tokens expire 6 hours after generation.
    		// You may override this by using token.setTtl(long millis).
    		AccessToken token = new AccessToken(LIVEKIT_API_KEY, LIVEKIT_API_SECRET);
    		token.setName(participantName);
    		token.setIdentity(participantName);

    		JSONObject metadata = new JSONObject();
    		metadata.put("livekitUrl", LIVEKIT_URL);
    		metadata.put("user", user.name);
    		metadata.put("role", user.role);
    		// add metadata to the token, which will be available in the participant's metadata
    		token.setMetadata(metadata.toString());
    		token.addGrants(new RoomJoin(true), new RoomName(roomName), new CanPublish(canPublish), new CanSubscribe(true));

    		response.put("token", token.toJwt());
    		return new ResponseEntity<>(response, HttpStatus.OK);
    	}

    	private boolean isUserLogged(HttpSession httpSession) {
    		return httpSession != null && httpSession.getAttribute("loggedUser") != null;
    	}
    }

    ```

Once the server application has returned the token, the client application will connect to the session using the token and it will decide which participants can publish their tracks based on their role.

```javascript hl_lines="10-16"
function joinRoom() {
  // ...

  getToken(myRoomName, myParticipantName).then((token) => {
    const livekitUrl = getLivekitUrlFromMetadata(token);

    room
      .connect(livekitUrl, token)
      .then(() => {
        const canPublish = room.localParticipant.permissions.canPublish;
        if (canPublish) {
          // Publish the tracks to the room
        } else {
          // Initialize the view for a subscriber
          initMainVideoThumbnail();
        }

        // ...
      })
      .catch((error) => {
        console.error("Error connecting to room", error);
      });
  });
}
```

## Deploying openvidu-roles (TODO)

=== ":fontawesome-brands-node-js:{.icon .lg-icon .tab-icon} Node"

    <h3> 1. Build the docker image</h3>

    Under the root project folder, you can see the `openvidu-roles-node/docker/` directory. Here it is included all the required files yo make it possible the deployment with OpenVidu.

    First of all, you will need to create the **openvidu-roles-node** docker image. Under `openvidu-roles-node/docker/` directory you will find the `create_image.sh` script. This script will create the docker image with the openvidu-roles-node server and the static files.


    ```bash
    ./create_image.sh openvidu/openvidu-roles-node-demo:X.Y.Z
    ```

    This script will create an image named `openvidu/openvidu-roles-node-demo:X.Y.Z`. This name will be used in the next step.

=== ":fontawesome-brands-java:{.icon .lg-icon .tab-icon} Java"

    <h3> 1. Build the docker image</h3>

    Under the root project folder, you can see the `openvidu-roles-java/docker/` directory. Here it is included all the required files yo make it possible the deployment with OpenVidu.

    First of all, you will need to create the **openvidu-roles-java** docker image. Under `openvidu-roles-java/docker/` directory you will find the `create_image.sh` script. This script will create the docker image with the openvidu-roles-java server and the static files.


    ```bash
    ./create_image.sh openvidu/openvidu-roles-java-demo:X.Y.Z
    ```

    This script will create an image named `openvidu/openvidu-roles-java-demo:X.Y.Z`. This name will be used in the next step.

<h3> 2. Deploy the docker image </h3>

Time to deploy the docker image. You can follow the [Deploy OpenVidu based application with Docker](/deployment/deploying-openvidu-apps/#with-docker) guide for doing this.
