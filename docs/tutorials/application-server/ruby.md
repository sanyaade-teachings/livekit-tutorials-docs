# Ruby

[Source code :simple-github:](https://github.com/OpenVidu/openvidu-livekit-tutorials/tree/master/application-server/ruby){ .md-button target=\_blank }

This is a minimal server application built for Ruby with [Sinatra](https://sinatrarb.com/){:target="\_blank"} that allows generating LiveKit tokens on demand.

It internally uses [LiveKit Ruby SDK](https://github.com/livekit/server-sdk-ruby){:target="\_blank"}.

## Running this application

#### Prerequisites

To run this application you will need **Ruby**:

- [Ruby](https://www.ruby-lang.org/en/downloads/){:target="\_blank"}

#### Download repository

```bash
git clone https://github.com/OpenVidu/openvidu-livekit-tutorials.git
```

#### Install dependencies

```bash
cd openvidu-livekit-tutorials/application-server/ruby
bundle install
```

#### Run application

```bash
ruby app.rb
```

## Understanding the code

The application is a simple Ruby app using the popular Sinatra web library. It has a single controller file `app.rb` that exports a unique endpoint:

- `/token` : generate a token for a given Room name and Participant name.

Let's see the code of the `app.rb` file:

```ruby title="<a href='https://github.com/OpenVidu/openvidu-livekit-tutorials/blob/master/application-server/ruby/app.rb#L1-L15' target='_blank'>app.rb</a>" linenums="1"
require "sinatra"
require "sinatra/cors"
require "livekit" # (1)!
require "./env.rb"

SERVER_PORT = ENV["SERVER_PORT"] || 6080 # (2)!
LIVEKIT_API_KEY = ENV["LIVEKIT_API_KEY"] || "devkey" # (3)!
LIVEKIT_API_SECRET = ENV["LIVEKIT_API_SECRET"] || "secret" # (4)!

set :port, SERVER_PORT # (5)!

register Sinatra::Cors # (6)!
set :allow_origin, "*"
set :allow_methods, "POST,OPTIONS"
set :allow_headers, "content-type"
```

1. Import `livekit` library
2. The port where the application will be listening
3. The API key of LiveKit Server
4. The API secret of LiveKit Server
5. Configure the port
6. Enable CORS support

The `app.rb` file imports the required dependencies and loads the necessary environment variables (defined in `env.rb` file):

- `SERVER_PORT`: the port where the application will be listening.
- `LIVEKIT_API_KEY`: the API key of LiveKit Server.
- `LIVEKIT_API_SECRET`: the API secret of LiveKit Server.

Finally the application configures the port and sets the CORS configuration for Sinatra.

#### Create token endpoint

The unique endpoint of the application is `/token`. It receives a JSON object with the following fields:

- `roomName`: the name of the Room where the user wants to connect.
- `participantName`: the name of the participant that wants to connect to the Room.

```ruby title="<a href='https://github.com/OpenVidu/openvidu-livekit-tutorials/blob/master/application-server/ruby/app.rb#L17-L34' target='_blank'>app.rb</a>" linenums="17"
post "/token" do
  content_type :json

  body = JSON.parse(request.body.read)
  room_name = body["roomName"]
  participant_name = body["participantName"]

  if room_name.nil? || participant_name.nil?
    status 400
    return JSON.generate("roomName and participantName are required")
  end

  token = LiveKit::AccessToken.new(api_key: LIVEKIT_API_KEY, api_secret: LIVEKIT_API_SECRET) # (1)!
  token.identity = participant_name # (2)!
  token.add_grant(roomJoin: true, room: room_name) # (3)!

  return JSON.generate(token.to_jwt) # (4)!
end
```

1. A new `AccessToken` is created providing the `LIVEKIT_API_KEY` and `LIVEKIT_API_SECRET`.
2. We set participant's identity in the AccessToken.
3. We set the video grants in the AccessToken. `roomJoin` allows the user to join a room and `room` determines the specific room. Check out all [Video Grants](https://docs.livekit.io/realtime/concepts/authentication/#Video-grant){:target="\_blank"}.
4. Finally, we convert the AccessToken to a JWT token and send it back to the client.

The endpoint first obtains the `roomName` and `participantName` parameters from the request body. If they are not available, it returns a `400` error.

If required fields are available, a new JWT token is created. For that we use the [LiveKit Ruby SDK](https://github.com/livekit/server-sdk-ruby){:target="\_blank"}:

1. A new `AccessToken` is created providing the `LIVEKIT_API_KEY` and `LIVEKIT_API_SECRET`.
2. We set participant's name and identity in the AccessToken.
3. We set the video grants in the AccessToken. `roomJoin` allows the user to join a room and `room` determines the specific room. Check out all [Video Grants](https://docs.livekit.io/realtime/concepts/authentication/#Video-grant){:target="\_blank"}.
4. Finally, we convert the AccessToken to a JWT token and send it back to the client.
