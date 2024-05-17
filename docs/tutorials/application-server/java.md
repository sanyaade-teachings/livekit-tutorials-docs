# Java

[Source code :simple-github:](https://github.com/OpenVidu/openvidu-livekit-tutorials/tree/master/application-server/java){ .md-button target=\_blank }

This is a minimal server application built for Java with [Spring Boot](https://spring.io/){:target="\_blank"} that allows:

- Generating LiveKit tokens on demand for any [application client](../../application-client/).
- Receiving LiveKit [webhook events](https://docs.livekit.io/realtime/server/webhooks/){target=\_blank}.

It internally uses [LiveKit Kotlin SDK](https://github.com/livekit/server-sdk-kotlin){:target="\_blank"}.

## Running this application

#### Prerequisites

To run this application you will need **Java** and **Maven**:

- [Java](https://www.java.com/en/download/manual.jsp){:target="\_blank"}
- [Maven](https://maven.apache.org){:target="\_blank"}

#### Download repository

```bash
git clone https://github.com/OpenVidu/openvidu-livekit-tutorials.git
```

#### Run application

```bash
cd openvidu-livekit-tutorials/application-server/java
mvn spring-boot:run
```

!!! info

    You can run any [Application Client](../../application-client/) to test against this server right away.

## Understanding the code

The application is a simple Go app with a single controller `Controller.java` that exports two endpoints:

- `/token` : generate a token for a given Room name and Participant name.
- `/webhook` : receive LiveKit webhook events.

Let's see the code of the `Controller.java` file:

```java title="<a href='https://github.com/OpenVidu/openvidu-livekit-tutorials/blob/master/application-server/java/src/main/java/io/openvidu/basic/java/Controller.java#L19-L27' target='_blank'>Controller.java</a>" linenums="19"
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

- `@CrossOrigin(origins = "*")`: allows the application to be accessed from any domain.
- `@RestController`: marks the class as a controller where every method returns a domain object instead of a view.

Going deeper, the `Controller` class has the following fields:

- `LIVEKIT_API_KEY`: the API key of LiveKit Server. It is injected from the environment variable `LIVEKIT_API_KEY` using the `@Value("${LIVEKIT_API_KEY}")` annotation.
- `LIVEKIT_API_SECRET`: the API secret of LiveKit Server. It is injected from the environment variable `LIVEKIT_API_SECRET` using the `@Value("${LIVEKIT_API_SECRET}")` annotation.

---

#### Create token

The endpoint `/token` accepts `POST` requests with a payload of type `application/json`, containing the following fields:

- `roomName`: the name of the Room where the user wants to connect.
- `participantName`: the name of the participant that wants to connect to the Room.

```java title="<a href='https://github.com/OpenVidu/openvidu-livekit-tutorials/blob/master/application-server/java/src/main/java/io/openvidu/basic/java/Controller.java#L33-L48' target='_blank'>Controller.java</a>" linenums="33"
@PostMapping(value = "/token", consumes = "application/json", produces = "application/json")
public ResponseEntity<String> createToken(@RequestBody Map<String, String> params) {
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

---

#### Receive webhook

The endpoint `/webhook` accepts `POST` requests with a payload of type `application/webhook+json`. This is the endpoint where LiveKit Server will send [webhook events](https://docs.livekit.io/realtime/server/webhooks/#Events){:target="\_blank"}.

```java title="<a href='https://github.com/OpenVidu/openvidu-livekit-tutorials/blob/master/application-server/java/src/main/java/io/openvidu/basic/java/Controller.java#L50-L60' target='_blank'>Controller.java</a>" linenums="50"
@PostMapping(value = "/webhook", consumes = "application/webhook+json")
public ResponseEntity<String> receiveWebhook(@RequestHeader("Authorization") String authHeader, @RequestBody String body) { // (1)!
	WebhookReceiver webhookReceiver = new WebhookReceiver(LIVEKIT_API_KEY, LIVEKIT_API_SECRET); // (2)!
	try {
		LivekitWebhook.WebhookEvent event = webhookReceiver.receive(body, authHeader); // (3)!
		System.out.println("LiveKit Webhook: " + event.toString());	// (4)!
	} catch (Exception e) {
		System.err.println("Error validating webhook event: " + e.getMessage());
	}
	return ResponseEntity.ok("ok");
}
```

1. We need the 'Authorization' header and the raw body of the HTTP request.
2. Initialize the WebhookReceiver using the `LIVEKIT_API_KEY` and `LIVEKIT_API_SECRET`. It will help validating and decoding incoming [webhook events](https://docs.livekit.io/realtime/server/webhooks/).
3. Obtain the `WebhookEvent` object using the `WebhookReceiver#receive` method. It takes the raw body as a String and the Authorization header of the request.
4. Consume the event as you whish.

We declare the 'Authorization' header and the raw body of the HTTP request as parameters of the our method. We need both of them to validate and decode the incoming webhook event.

Then we initialize a `WebhookReceiver` object using the `LIVEKIT_API_KEY` and `LIVEKIT_API_SECRET`.

Finally we obtain a `WebhookEvent` object calling method `WebhookReceiver#receive`. It takes the raw body as a String and the `Authorization` header of the request. If everything is correct, you can do whatever you want with the event (in this case, we just log it).

Remember to return a `200` OK response at the end to let LiveKit Server know that the webhook was received correctly.

<br>