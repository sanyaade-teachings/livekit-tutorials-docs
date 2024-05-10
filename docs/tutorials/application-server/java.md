# Java

[Source code :simple-github:](https://github.com/OpenVidu/openvidu-livekit-tutorials/tree/master/application-server/java){ .md-button target=\_blank }

This is a minimal server application built for Java with [Spring Boot](https://spring.io/){:target="\_blank"} that allows generating LiveKit tokens on demand.

It internally uses [LiveKit Kotlin SDK](https://github.com/livekit/server-sdk-kotlin){:target="\_blank"}.

## Running this application

#### Prerequisites

To run this application you will need **Java** and **Maven**:

- [Java](https://www.java.com/en/download/manual.jsp){:target="\_blank"}
- [Maven](https://maven.apache.org){:target="\_blank"}

#### Download repository

```bash
git clone https://github.com/OpenVidu/openvidu-livekit-tutorials.git
cd openvidu-livekit-tutorials/application-server/java
```

#### Run application

```bash
mvn spring-boot:run
```

## Understanding the code

The application is a simple Go app with a single controller `Controller.java` that exports a unique endpoint:

- `/token` : Generate a token for a given Room name and Participant name

Let's see the code of the `Controller.java` file:

```java title="<a href='https://github.com/OpenVidu/openvidu-livekit-tutorials/blob/master/application-server/java/src/main/java/io/openvidu/basic/java/Controller.java#L14-L22' target='_blank'>Controller.java</a>" linenums="14"
@CrossOrigin(origins = "*") // (1)!
@RestController // (2)!
public class Controller {

	@Value("${livekit.api.key}")
	private String LIVEKIT_API_KEY; // (3)!

	@Value("${livekit.api.secret}")
	private String LIVEKIT_API_SECRET; // (4)!

	...
}
```

1. Allows the application to be accessed from any domain
2. Marks the class as a controller where every method returns a domain object instead of a view
3. The API key of LiveKit Server
4. The API secret of LiveKit Server

Starting by the top, the `Controller` class has the following annotations:

- `@CrossOrigin(origins = "*")`: Allows the application to be accessed from any domain.
- `@RestController`: Marks the class as a controller where every method returns a domain object instead of a view.

Going deeper, the `Controller` class has the following fields:

- `LIVEKIT_API_KEY`: the API key of LiveKit Server. It is injected from the environment variable `LIVEKIT_API_KEY` using the `@Value("${LIVEKIT_API_KEY}")` annotation.
- `LIVEKIT_API_SECRET`: the API secret of LiveKit Server. It is injected from the environment variable `LIVEKIT_API_SECRET` using the `@Value("${LIVEKIT_API_SECRET}")` annotation.

#### Create token endpoint

The unique endpoint of the application is `/token`. It receives a JSON object in the request body with the following fields:

- `roomName`: the name of the Room where the user wants to connect.
- `participantName`: the name of the participant that wants to connect to the Room.

```java title="<a href='https://github.com/OpenVidu/openvidu-livekit-tutorials/blob/master/application-server/java/src/main/java/io/openvidu/basic/java/Controller.java#L28-L43' target='_blank'>Controller.java</a>" linenums="28"
@PostMapping(value = "/token", produces = "application/json")
public ResponseEntity<String> getToken(@RequestBody Map<String, String> params) {
	String roomName = params.get("roomName");
	String participantName = params.get("participantName");

	if (roomName == null || participantName == null) {
		return ResponseEntity.badRequest().body("\"roomName and participantName are required\"");
	}

	AccessToken token = new AccessToken(LIVEKIT_API_KEY, LIVEKIT_API_SECRET); // (1)!
	token.setName(participantName); // (2)!
	token.setIdentity(participantName);
	token.addGrants(new RoomJoin(true), new RoomName(roomName)); // (3)!

	return ResponseEntity.ok("\"" + token.toJwt() + "\""); // (4)!
}
```

1. A new `AccessToken` is created providing the `LIVEKIT_API_KEY` and `LIVEKIT_API_SECRET`.
2. We set participant's name and identity in the AccessToken.
3. We set the video grants in the AccessToken. `RoomJoin` allows the user to join a room and `RoomName` determines the specific room. Check out all [Video Grants](https://docs.livekit.io/realtime/concepts/authentication/#Video-grant){:target="\_blank"}.
4. Finally, the token is sent back to the client.

The endpoint first obtains the `roomName` and `participantName` parameters from the request body. If they are not available, it returns a `400` error.

If required fields are available, a new JWT token is created. For that we use the [LiveKit Kotlin SDK](https://github.com/livekit/server-sdk-kotlin){:target="\_blank"}:

1. A new `AccessToken` is created providing the `LIVEKIT_API_KEY` and `LIVEKIT_API_SECRET`.
2. We set participant's name and identity in the AccessToken.
3. We set the video grants in the AccessToken. `RoomJoin` allows the user to join a room and `RoomName` determines the specific room. Check out all [Video Grants](https://docs.livekit.io/realtime/concepts/authentication/#Video-grant){:target="\_blank"}.
4. Finally, the token is sent back to the client.
