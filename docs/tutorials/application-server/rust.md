# Rust

[Source code :simple-github:](https://github.com/OpenVidu/openvidu-livekit-tutorials/tree/master/application-server/rust){ .md-button target=\_blank }

This is a minimal server application built for Rust with [Axum](https://github.com/tokio-rs/axum){:target="\_blank"} that allows generating LiveKit tokens on demand.

It internally uses the [LiveKit Rust SDK](https://github.com/livekit/rust-sdks){:target="\_blank"}.

## Running this application

#### Prerequisites

To run this application you will need **Rust**:

- [Rust](https://www.rust-lang.org/tools/install){:target="\_blank"}

#### Download repository

```bash
git clone https://github.com/OpenVidu/openvidu-livekit-tutorials.git
```

#### Run application

```bash
cd openvidu-livekit-tutorials/application-server/rust
cargo run
```

## Understanding the code

The application is a simple Rust app with a single file `main.rs` that exports a unique endpoint:

- `/token` : generate a token for a given Room name and Participant name.

Let's see the code of the `main.rs` file:

```rust title="<a href='https://github.com/OpenVidu/openvidu-livekit-tutorials/blob/master/application-server/rust/src/main.rs#L1-L34' target='_blank'>main.rs</a>" linenums="1"
use axum::{
    extract::Json, http::header::CONTENT_TYPE, http::Method, http::StatusCode, routing::post,
    Router,
};
use dotenv::dotenv;
use env_logger::Env;
use livekit_api::access_token; // (1)!
use log::info;
use serde_json::Value;
use std::env;
use tower_http::cors::{Any, CorsLayer};

#[tokio::main]
async fn main() {
    env_logger::Builder::from_env(Env::default().default_filter_or("info")).init(); // Init logger
    dotenv().ok(); // Load environment variables from .env

    // Check that required environment variables are set
    let server_port = env::var("SERVER_PORT").unwrap_or("6080".to_string());
    env::var("LIVEKIT_API_KEY").expect("LIVEKIT_API_KEY is not set"); // (2)!
    env::var("LIVEKIT_API_SECRET").expect("LIVEKIT_API_SECRET is not set"); // (3)!

    let cors = CorsLayer::new() // (4)!
        .allow_methods([Method::POST])
        .allow_origin(Any)
        .allow_headers([CONTENT_TYPE]);

    let app = Router::new().route("/token", post(create_token)).layer(cors); // (5)!

    let listener = tokio::net::TcpListener::bind("0.0.0.0:".to_string() + &server_port)
        .await
        .unwrap();
    axum::serve(listener, app).await.unwrap(); // (6)!
}
```

1. Import the Rust LiveKit library.
2. The API key of LiveKit Server.
3. The API secret of LiveKit Server.
4. Enable CORS support.
5. Define the `/token` endpoint.
6. Start the server listening on the specified port.

The `main.rs` file imports the required dependencies and loads the necessary environment variables:

- `SERVER_PORT`: the port where the application will be listening.
- `LIVEKIT_API_KEY`: the API key of LiveKit Server.
- `LIVEKIT_API_SECRET`: the API secret of LiveKit Server.

Finally the `axum` application is initialized.

#### Create token endpoint

The unique endpoint of the application is `/token`. It receives a JSON object with the following fields:

- `roomName`: the name of the Room where the user wants to connect.
- `participantName`: the name of the participant that wants to connect to the Room.

```go title="<a href='https://github.com/OpenVidu/openvidu-livekit-tutorials/blob/master/application-server/rust/src/main.rs#L36-L70' target='_blank'>main.rs</a>" linenums="36"
async fn create_token(payload: Option<Json<Value>>) -> (StatusCode, Json<String>) {
    if let Some(payload) = payload {
        let livekit_api_key = env::var("LIVEKIT_API_KEY").expect("LIVEKIT_API_KEY is not set");
        let livekit_api_secret =
            env::var("LIVEKIT_API_SECRET").expect("LIVEKIT_API_SECRET is not set");

        let room_name = payload.get("roomName").expect("roomName is required");
        let participant_name = payload
            .get("participantName")
            .expect("participantName is required");

        let token = access_token::AccessToken::with_api_key(&livekit_api_key, &livekit_api_secret) // (1)!
            .with_identity(&participant_name.to_string()) // (2)!
            .with_name(&participant_name.to_string())
            .with_grants(access_token::VideoGrants { // (3)!
                room_join: true,
                room: room_name.to_string(),
                ..Default::default()
            })
            .to_jwt() // (4)!
            .unwrap();

        info!(
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
