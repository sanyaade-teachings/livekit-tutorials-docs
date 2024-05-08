# openvidu-basic-ruby

[Source code :simple-github:](https://github.com/OpenVidu/openvidu-livekit-tutorials){ .md-button target=_blank }

This is a minimal server application sample built for Ruby with [Sinatra](https://sinatrarb.com/){:target="_blank"}.
It internally uses the [livekit-ruby-server-sdk](https://github.com/livekit/server-sdk-ruby){:target="_blank"}.


## Running this application

#### Prerequisites
To run this application you will need **Ruby** installed on your system:

- [Ruby](https://www.ruby-lang.org/en/downloads/){:target="_blank"}

#### Download repository

```bash
git clone https://github.com/OpenVidu/openvidu-livekit-tutorials.git
cd openvidu-livekit-tutorials/openvidu-basic-ruby
```

#### Install dependencies

```bash
bundle install
```

#### Run application

```bash
ruby app.rb
```

## Understanding the code

The application is a simple Ruby application using the popular Sinatra (web server) and Faraday (http client) libraries. It has a single controller file `app.rb` that exports a unique endpoint:

- `/token` : Generate a token for a given Room name and Participant name

Let's see the code of the controller:

```ruby
require 'sinatra'
require 'sinatra/cors'
require 'faraday'
require 'json'
require 'livekit'
require './env.rb'

# Load env variables
SERVER_PORT = ENV['SERVER_PORT']
LIVEKIT_API_KEY = ENV['LIVEKIT_API_KEY']
LIVEKIT_API_SECRET = ENV['LIVEKIT_API_SECRET']

set :port, SERVER_PORT

register Sinatra::Cors
set :allow_origin, "*"
set :allow_methods, "POST,OPTIONS"
set :allow_headers, "content-type"
```

Starting by the top, the `app.rb` file has the following statements:

- All the `require` instructions: load the necessary libraries and environment variables.


Then the application configures the port retrieved from the environment variables (file `env.rb`) and sets the CORS configuration for Sinatra. The CORS policy is configured to allow requests from any origin and any header.

<br>

#### Create token endpoint

The unique endpoint of the application is `/token`. It receives a JSON object with the following fields:

- `roomName`: the name of the Room where the user wants to connect.
- `participantName`: the name of the participant that wants to connect to the Room.

```ruby
post '/token' do

  content_type :json

  body = JSON.parse(request.body.read)
  room_name = body['roomName']
  participant_name = body['participantName']

  if room_name.nil? || participant_name.nil?
    status 400
    return 'roomName and participantName are required'
  end

  # By default, a token expires after 6 hours.
  # You may override this by passing in ttl when creating the token. ttl is expressed in seconds.
  token = LiveKit::AccessToken.new(api_key: LIVEKIT_API_KEY, api_secret: LIVEKIT_API_SECRET)
  token.identity = participant_name
  token.name = participant_name
  token.add_grant(roomJoin: true, room: room_name)

  token.to_jwt
end
```

The endpoint first checks if the request body has the `roomName` and `participantName` fields. If not, it returns a `400` error.

Then, it creates a new token using the [LiveKit Ruby SDK](https://github.com/livekit/server-sdk-ruby){:target="_blank"}.

The token is created with the `LIVEKIT_API_KEY` and `LIVEKIT_API_SECRET` environment variables. This object will be used to create a new token for the user.

After that, we set the `identity` and `name` fields of the `AccessToken` object with the `participantName` received from the request body.

Then, we add a new grant to the token. The grant is a `roomJoin` grant with the `room` field set to the `roomName` received from the request body.

Finally, the `AccessToken` is converted to a JWT token and returned in the response body.
