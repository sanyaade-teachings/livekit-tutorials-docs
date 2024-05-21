# Rust

[Source code :simple-github:](https://github.com/OpenVidu/openvidu-livekit-tutorials/tree/master/application-server/rust){ .md-button target=\_blank }

This is a minimal server application built for Rust with [Axum](https://github.com/tokio-rs/axum){:target="\_blank"} that allows:

- Generating LiveKit tokens on demand for any [application client](../../application-client/).
- Receiving LiveKit [webhook events](https://docs.livekit.io/realtime/server/webhooks/){target=\_blank}.

It internally uses the [LiveKit Rust SDK](https://github.com/livekit/rust-sdks){:target="\_blank"}.

## Running this application

Download the tutorial code:

```bash
git clone https://github.com/OpenVidu/openvidu-livekit-tutorials.git
```

--8<-- "docs/tutorials/shared/rust.md"

!!! info

    You can run any [Application Client](../../application-client/) to test against this server right away.

## Understanding the code

The application is a simple Rust app with a single file `main.rs` that exports two endpoints:

- `/token` : generate a token for a given Room name and Participant name.
- `/webhook` : receive LiveKit webhook events.

Let's see the code of the `main.rs` file:

```rust title="<a href='https://github.com/OpenVidu/openvidu-livekit-tutorials/blob/master/application-server/rust/src/main.rs#L1-L38' target='_blank'>main.rs</a>" linenums="1"
use axum::http::HeaderMap;
use axum::{
    extract::Json, http::header::CONTENT_TYPE, http::Method, http::StatusCode, routing::post,
    Router,
};
use dotenv::dotenv;
use livekit_api::access_token::AccessToken; // (1)!
use livekit_api::access_token::TokenVerifier;
use livekit_api::access_token::VideoGrants;
use livekit_api::webhooks::WebhookReceiver;
use serde_json::Value;
use std::env;
use tower_http::cors::{Any, CorsLayer};

#[tokio::main]
async fn main() {
    dotenv().ok(); // Load environment variables from .env

    // Check that required environment variables are set
    let server_port = env::var("SERVER_PORT").unwrap_or("6080".to_string());
    env::var("LIVEKIT_API_KEY").expect("LIVEKIT_API_KEY is not set"); // (2)!
    env::var("LIVEKIT_API_SECRET").expect("LIVEKIT_API_SECRET is not set"); // (3)!

    let cors = CorsLayer::new() // (4)!
        .allow_methods([Method::POST])
        .allow_origin(Any)
        .allow_headers([CONTENT_TYPE]);

    let app = Router::new() // (5)!
        .route("/token", post(create_token))
        .route("/webhook", post(receive_webhook))
        .layer(cors);

    let listener = tokio::net::TcpListener::bind("0.0.0.0:".to_string() + &server_port)
        .await
        .unwrap();
    axum::serve(listener, app).await.unwrap(); // (6)!
}
```

1. Import all necessary dependencies from the Rust LiveKit library.
2. The API key of LiveKit Server.
3. The API secret of LiveKit Server.
4. Enable CORS support.
5. Define `/token` and `/weebhook` endpoints.
6. Start the server listening on the specified port.

The `main.rs` file imports the required dependencies and loads the necessary environment variables:

- `SERVER_PORT`: the port where the application will be listening.
- `LIVEKIT_API_KEY`: the API key of LiveKit Server.
- `LIVEKIT_API_SECRET`: the API secret of LiveKit Server.

Then CORS support is enabled and the endpoints are defined. Finally the `axum` application is initialized on the specified port.

---

#### Create token endpoint

The endpoint `/token` accepts `POST` requests with a payload of type `application/json`, containing the following fields:

- `roomName`: the name of the Room where the user wants to connect.
- `participantName`: the name of the participant that wants to connect to the Room.

```rust title="<a href='https://github.com/OpenVidu/openvidu-livekit-tutorials/blob/master/application-server/rust/src/main.rs#L40-L74' target='_blank'>main.rs</a>" linenums="40"
async fn create_token(payload: Option<Json<Value>>) -> (StatusCode, Json<String>) {
    if let Some(payload) = payload {
        let livekit_api_key = env::var("LIVEKIT_API_KEY").expect("LIVEKIT_API_KEY is not set");
        let livekit_api_secret =
            env::var("LIVEKIT_API_SECRET").expect("LIVEKIT_API_SECRET is not set");

        let room_name = payload.get("roomName").expect("roomName is required");
        let participant_name = payload
            .get("participantName")
            .expect("participantName is required");

        let token = AccessToken::with_api_key(&livekit_api_key, &livekit_api_secret) // (1)!
            .with_identity(&participant_name.to_string()) // (2)!
            .with_name(&participant_name.to_string())
            .with_grants(VideoGrants { // (3)!
                room_join: true,
                room: room_name.to_string(),
                ..Default::default()
            })
            .to_jwt() // (4)!
            .unwrap();

        println!(
            "Sending token for room {} to participant {}",
            room_name, participant_name
        );

        return (StatusCode::OK, Json(token)); // (5)!
    } else {
        return (
            StatusCode::BAD_REQUEST,
            Json("roomName and participantName are required".to_string()),
        );
    }
}
```

1. A new `AccessToken` is created providing the `LIVEKIT_API_KEY` and `LIVEKIT_API_SECRET`.
2. We set participant's name and identity in the AccessToken.
3. We set the video grants in the AccessToken. `room_joim` allows the user to join a room and `room` determines the specific room. Check out all [Video Grants](https://docs.livekit.io/realtime/concepts/authentication/#Video-grant){:target="\_blank"}.
4. We convert the AccessToken to a JWT token.
5. Finally, the token is sent back to the client.

The endpoint first obtains the `roomName` and `participantName` parameters from the request body. If they are not available, it returns a `400` error.

If required fields are available, a new JWT token is created. For that we use the [LiveKit Rust SDK](https://github.com/livekit/rust-sdks){:target="\_blank"}:

1. A new `AccessToken` is created providing the `LIVEKIT_API_KEY` and `LIVEKIT_API_SECRET`.
2. We set participant's name and identity in the AccessToken.
3. We set the video grants in the AccessToken. `room_joim` allows the user to join a room and `room` determines the specific room. Check out all [Video Grants](https://docs.livekit.io/realtime/concepts/authentication/#Video-grant){:target="\_blank"}.
4. We convert the AccessToken to a JWT token.
5. Finally, the token is sent back to the client.

---

#### Receive webhook

The endpoint `/webhook` accepts `POST` requests with a payload of type `application/webhook+json`. This is the endpoint where LiveKit Server will send [webhook events](https://docs.livekit.io/realtime/server/webhooks/#Events){:target="\_blank"}.

```rust title="<a href='https://github.com/OpenVidu/openvidu-livekit-tutorials/blob/master/application-server/rust/src/main.rs#L76-L97' target='_blank'>main.rs</a>" linenums="76"
async fn receive_webhook(headers: HeaderMap, body: String) -> StatusCode {
    let livekit_api_key = env::var("LIVEKIT_API_KEY").expect("LIVEKIT_API_KEY is not set");
    let livekit_api_secret = env::var("LIVEKIT_API_SECRET").expect("LIVEKIT_API_SECRET is not set");
    let token_verifier = TokenVerifier::with_api_key(&livekit_api_key, &livekit_api_secret); // (1)!
    let webhook_receiver = WebhookReceiver::new(token_verifier); // (2)!

    let auth_header: &str = headers // (3)!
        .get("Authorization")
        .expect("Authorization header is required")
        .to_str()
        .unwrap();

    let res = webhook_receiver.receive(&body, auth_header); // (4)!

    if let Ok(event) = res {
        println!("LiveKit WebHook: {:?}", event); // (5)!
        return StatusCode::OK;
    } else {
        println!("Error validating webhook event: {:?}", res);
        return StatusCode::UNAUTHORIZED;
    }
}
```

1. Create a `TokenVerifier` with the `LIVEKIT_API_KEY` and `LIVEKIT_API_SECRET`. This will validate the webhook event to confirm it is actually coming from our LiveKit Server.
2. Create a `WebhookReceiver` with the `TokenVerifier`.
3. Get the `Authorization` header from the HTTP request.
4. Obtain the webhook event using the `WebhookReceiver#receive` method. It expects the raw string body of the request and the `Authorization` header.
5. Consume the event as you wish.

We declare as function parameters the map of headers (`#!rust headers: HeaderMap`) and the raw body (`#!rust body: String`) of the HTTP request. We will need both of them to validate and decode the incoming webhook event. We then:

1. Create a `TokenVerifier` with the `LIVEKIT_API_KEY` and `LIVEKIT_API_SECRET`. This will validate the webhook event to confirm it is actually coming from our LiveKit Server.
2. Create a `WebhookReceiver` with the `TokenVerifier`.
3. Get the `Authorization` header from the HTTP request.
4. Obtain the webhook event using the `WebhookReceiver#receive` method. It expects the raw string body of the request and the `Authorization` header.
5. Consume the event as you wish (in this case, we just log it).

Remember to return a `200` OK response at the end to let LiveKit Server know that the webhook was received correctly.

<br>