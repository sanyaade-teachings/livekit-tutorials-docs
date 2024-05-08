
# openvidu-basic-java

[Source code :simple-github:](https://github.com/OpenVidu/openvidu-livekit-tutorials){ .md-button target=_blank }


This is a minimal server application sample built for Java with [Spring Boot](https://spring.io/){:target="_blank"}.
It internally uses [livekit-kotlin-server-sdk](https://github.com/livekit/server-sdk-kotlin){:target="_blank"}.

## Running this application

#### Prerequisites
To run this application you will need **Java** and **Maven**:

- [Java (>=11)](https://www.java.com/en/download/manual.jsp){:target="_blank"}
- [Maven](https://maven.apache.org){:target="_blank"}

#### Download repository

```bash
git clone https://github.com/OpenVidu/openvidu-livekit-tutorials.git
cd openvidu-livekit-tutorials/openvidu-basic-java
```

#### Run application

```bash
mvn spring-boot:run
```

## Understanding the code

The application is a simple Spring Boot application with a single controller class `Controller.java` that exports a unique endpoint:

- `/token` : Generate a token for a given Room name and Participant name

Let's see the code of the controller:

```java
@CrossOrigin(origins = "*")
@RestController
public class Controller {

	@Value("${LIVEKIT_API_KEY}")
	private String LIVEKIT_API_KEY;

	@Value("${LIVEKIT_API_SECRET}")
	private String LIVEKIT_API_SECRET;

	...
}
```

Starting by the top, the `Controller` class has the following annotations:

- `@CrossOrigin(origins = "*")`: Allows the application to be accessed from any domain.
- `@RestController`: Marks the class as a controller where every method returns a domain object instead of a view.

Going deeper, the `Controller` class has the following fields:

- `LIVEKIT_API_KEY`: the API key of the LiveKit deployment. It is injected from the environment variable `LIVEKIT_API_KEY` using the `@Value("${LIVEKIT_API_KEY}")` annotation.

- `LIVEKIT_API_SECRET`: the API secret of the LiveKit deployment. It is injected from the environment variable `LIVEKIT_API_SECRET` using the `@Value("${LIVEKIT_API_SECRET}")` annotation.


<br>

#### Create token endpoint

The unique endpoint of the application is `/token`. It receives a JSON object in the request body with the following fields:

- `roomName`: the name of the Room where the user wants to connect.
- `participantName`: the name of the participant that wants to connect to the Room.

```java
@CrossOrigin(origins = "*")
@RestController
public class Controller {
	...

	/**
	 * @param params The JSON object with roomName and participantName
	 * @return The JWT token
	 */
	@PostMapping("/token")
	public ResponseEntity<String> getToken(@RequestBody(required = true) Map<String, String> params) {
		String roomName = params.get("roomName");
		String participantName = params.get("participantName");

		if(roomName == null || participantName == null) {
			return new ResponseEntity<>(HttpStatus.BAD_REQUEST);
		}

		// By default, tokens expire 6 hours after generation.
		// You may override this by using token.setTtl(long millis).
		AccessToken token = new AccessToken(LIVEKIT_API_KEY, LIVEKIT_API_SECRET);
		token.setName(participantName);
		token.setIdentity(participantName);
		token.addGrants(new RoomJoin(true), new RoomName(roomName));

		return new ResponseEntity<>(token.toJwt(), HttpStatus.OK);
	}
}
```

The endpoint first checks if the request body has the `roomName` and `participantName` fields. If not, it returns a `400` error.

Then, it creates a new `AccessToken` object with the `LIVEKIT_API_KEY` and `LIVEKIT_API_SECRET` environment variables. This object will be used to create a new token for the user.

After that, we set the `name` and `identity` fields of the `AccessToken` object with the `participantName` received from the request body.

Finally, we add the `RoomJoin` and `RoomName` grants to the `AccessToken` object. The `RoomJoin` grant allows the user to join the room, and the `RoomName` grant allows the user to join the room with the name specified in the `RoomName` object.

