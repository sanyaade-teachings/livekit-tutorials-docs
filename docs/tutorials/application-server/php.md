# PHP

[Source code :simple-github:](https://github.com/OpenVidu/openvidu-livekit-tutorials/tree/master/application-server/php){ .md-button target=\_blank }

This is a minimal server application built for PHP that allows generating LiveKit tokens on demand.

It internally uses [LiveKit PHP SDK](https://github.com/agence104/livekit-server-sdk-php){:target="\_blank"}.

## Running this application

#### Prerequisites

To run this application you will need **PHP** and **Composer**:

- [PHP](https://www.php.net/downloads)
- [Composer](https://getcomposer.org/download/)

#### Download repository

```bash
git clone https://github.com/OpenVidu/openvidu-livekit-tutorials.git
cd openvidu-livekit-tutorials/application-server/php
```

#### Install dependencies

```bash
composer install
```

#### Run application

```bash
composer start
```

## Understanding the code

The application is a simple PHP app with a single file `index.php` that exports a unique endpoint:

- `/token` : Generate a token for a given Room name and Participant name

Let's see the code of the `index.php` file:

```php title="<a href='https://github.com/OpenVidu/openvidu-livekit-tutorials/blob/master/application-server/php/index.php#L1-L18' target='_blank'>index.php</a>" linenums="1"
<?php
require __DIR__ . "/vendor/autoload.php";

use Agence104\LiveKit\AccessToken; // (1)!
use Agence104\LiveKit\AccessTokenOptions;
use Agence104\LiveKit\VideoGrant;
use Dotenv\Dotenv;

Dotenv::createImmutable(__DIR__)->safeLoad();

header("Access-Control-Allow-Origin: *"); // (2)!
header("Content-Type: application/json; charset=UTF-8");
header("Access-Control-Allow-Methods: OPTIONS,POST");
header("Access-Control-Max-Age: 3600");
header("Access-Control-Allow-Headers: Content-Type, Access-Control-Allow-Headers, Authorization, X-Requested-With");

$LIVEKIT_API_KEY = $_ENV["LIVEKIT_API_KEY"] ?? "devkey"; // (3)!
$LIVEKIT_API_SECRET = $_ENV["LIVEKIT_API_SECRET"] ?? "secret"; // (4)!
```

1. Import all necessary dependencies from the PHP livekit library.
2. Configure HTTP headers for the web server: enables CORS support, sets the content type to JSON, and allows only OPTIONS and POST requests.
3. The API key of LiveKit Server.
4. The API secret of LiveKit Server.

The `index.php` file imports the required dependencies, sets the HTTP headers for the web server and loads the necessary environment variables:

- `LIVEKIT_API_KEY`: the API key of LiveKit Server.
- `LIVEKIT_API_SECRET`: the API secret of LiveKit Server.

#### Create token endpoint

The unique endpoint of the application is `/token`. It receives a JSON object with the following fields:

- `roomName`: the name of the Room where the user wants to connect.
- `participantName`: the name of the participant that wants to connect to the Room.

```php title="<a href='https://github.com/OpenVidu/openvidu-livekit-tutorials/blob/master/application-server/php/index.php#L20-L44' target='_blank'>index.php</a>" linenums="19"
<?php
if ($_SERVER["REQUEST_METHOD"] === "POST" && $_SERVER["PATH_INFO"] === "/token") {
    $data = json_decode(file_get_contents("php://input"), true);

    $roomName = $data["roomName"] ?? null;
    $participantName = $data["participantName"] ?? null;

    if (!$roomName || !$participantName) {
        http_response_code(400);
        echo json_encode("roomName and participantName are required");
        exit();
    }

    $tokenOptions = (new AccessTokenOptions()) // (1)!
        ->setIdentity($participantName);
    $videoGrant = (new VideoGrant()) // (2)!
        ->setRoomJoin()
        ->setRoomName($roomName);
    $token = (new AccessToken($LIVEKIT_API_KEY, $LIVEKIT_API_SECRET)) // (3)!
        ->init($tokenOptions)
        ->setGrant($videoGrant)
        ->toJwt();

    echo json_encode($token); // (4)!
    exit();
}
```

1. Create an `AccessTokenOptions` object with the participant's identity.
2. Set the video grants in the token options. `setRoomJoin` allows the user to join a room and `setRoomName` determines the specific room. Check out all [Video Grants](https://docs.livekit.io/realtime/concepts/authentication/#Video-grant){:target="\_blank"}.
3. We create the `AccessToken` providing the `LIVEKIT_API_KEY` and `LIVEKIT_API_SECRET`, initialize it with the token options, set the video grants and generate the JWT token.
4. Finally, the token is sent back to the client.

The endpoint first obtains the `roomName` and `participantName` parameters from the request body. If they are not available, it returns a `400` error.

If required fields are available, a new JWT token is created. For that we use the [LiveKit PHP SDK](https://github.com/agence104/livekit-server-sdk-php){:target="\_blank"}:

1. Create an `AccessTokenOptions` object with the participant's identity.
2. Set the video grants in the token options. `setRoomJoin` allows the user to join a room and `setRoomName` determines the specific room. Check out all [Video Grants](https://docs.livekit.io/realtime/concepts/authentication/#Video-grant){:target="\_blank"}.
3. We create the `AccessToken` providing the `LIVEKIT_API_KEY` and `LIVEKIT_API_SECRET`, initialize it with the token options, set the video grants and generate the JWT token.
4. Finally, the token is sent back to the client.
