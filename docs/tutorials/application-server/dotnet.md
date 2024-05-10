# .NET

[Source code :simple-github:](https://github.com/OpenVidu/openvidu-livekit-tutorials/tree/master/application-server/dotnet){ .md-button target=\_blank }

This is a minimal server application built for .NET with [ASP.NET Core Minimal APIs](https://docs.microsoft.com/aspnet/core/tutorials/min-web-api?view=aspnetcore-6.0&tabs=visual-studio){:target="\_blank"} that allows generating LiveKit tokens on demand.

Unfortunately there is no .NET SDK for LiveKit available, so the application has to manually build LiveKit compatible JWT tokens using the .NET library `System.IdentityModel.Tokens.Jwt`. It is a fairly easy process.

## Running this application

#### Prerequisites

To run this application you will need **.NET**:

- [.NET 8.0](https://dotnet.microsoft.com/en-us/download){:target="\_blank"}

#### Download repository

```bash
git clone https://github.com/OpenVidu/openvidu-livekit-tutorials.git
cd openvidu-livekit-tutorials/application-server/dotnet
```

#### Run application

```bash
dotnet run
```

## Understanding the code

The application is a simple ASP.NET Core Minimal APIs app with a single file `Program.cs` that exports a unique endpoint:

- `/token` : Generate a token for a given Room name and Participant name

Let's see the code `Program.cs` file:

```cs title="<a href='https://github.com/OpenVidu/openvidu-livekit-tutorials/blob/master/application-server/dotnet/Program.cs#L6-L35' target='_blank'>Program.cs</a>" linenums="6"
var builder = WebApplication.CreateBuilder(args); // (1)!
var MyAllowSpecificOrigins = "_myAllowSpecificOrigins"; // (2)!

IConfiguration config = new ConfigurationBuilder() // (3)!
                .SetBasePath(Directory.GetCurrentDirectory())
                .AddJsonFile("appsettings.json")
                .AddEnvironmentVariables().Build();

// Load env variables
var SERVER_PORT = config.GetValue<int>("SERVER_PORT"); // (4)!
var LIVEKIT_API_KEY = config.GetValue<string>("LIVEKIT_API_KEY"); // (5)!
var LIVEKIT_API_SECRET = config.GetValue<string>("LIVEKIT_API_SECRET"); // (6)!

// Enable CORS support
builder.Services.AddCors(options => // (7)!
{
    options.AddPolicy(name: MyAllowSpecificOrigins,
                      builder =>
                      {
                          builder.WithOrigins("*").AllowAnyHeader();
                      });
});

builder.WebHost.UseKestrel(serverOptions => // (8)!
{
    serverOptions.ListenAnyIP(SERVER_PORT);
});

var app = builder.Build(); // (9)!
app.UseCors(MyAllowSpecificOrigins);
```

1. A `WebApplicationBuilder` instance to build the Flask application.
2. The name of the CORS policy to be used in the application.
3. A `IConfiguration` instance to load the configuration from the `appsettings.json` file, including the requried environment variables.
4. The port where the application will be listening
5. The API key of LiveKit Server
6. The API secret of LiveKit Server
7. Configure CORS support
8. Configure the port
9. Build the application and enable CORS support

The `Program.cs` file imports the required dependencies and loads the necessary environment variables (defined in `appsettings.json` file):

- `SERVER_PORT`: the port where the application will be listening.
- `LIVEKIT_API_KEY`: the API key of LiveKit Server.
- `LIVEKIT_API_SECRET`: the API secret of LiveKit Server.

Finally the application enables CORS support and the port where the application will be listening.

#### Create token endpoint

The unique endpoint of the application is `/token`. It receives a JSON object with the following fields:

- `roomName`: the name of the Room where the user wants to connect.
- `participantName`: the name of the participant that wants to connect to the Room.

```cs title="<a href='https://github.com/OpenVidu/openvidu-livekit-tutorials/blob/master/application-server/dotnet/Program.cs#L37-L52' target='_blank'>Program.cs</a>" linenums="37"
app.MapPost("/token", async (HttpRequest request) =>
{
    var body = new StreamReader(request.Body); // (1)!
    string postData = await body.ReadToEndAsync();
    Dictionary<string, dynamic> bodyParams = JsonSerializer.Deserialize<Dictionary<string, dynamic>>(postData) ?? new Dictionary<string, dynamic>();

    if (bodyParams.TryGetValue("roomName", out var roomName) && bodyParams.TryGetValue("participantName", out var participantName))
    {
        var token = CreateLiveKitJWT(roomName.ToString(), participantName.ToString()); // (2)!
        return Results.Json(token); // (3)!
    }
    else
    {
        return Results.BadRequest("\"roomName\" and \"participantName\" are required"); // (4)!
    }
});
```

1. The endpoint obtains a Dictionary from the body request, and check if fields `roomName` and `participantName` are available.
2. Create a new JWT token with the room and participant name.
3. Return the token to the client.
4. Return a `400` error if required fields are not available.

The endpoint obtains a Dictionary from the body request, and check if fields `roomName` and `participantName` are available. If not, it returns a `400` error.

If required fields are available, a new JWT token is created. Unfortunately there is no .NET SDK for LiveKit, so we need to create the JWT token manually. The `CreateLiveKitJWT` method is responsible for creating the LiveKit compatible JWT token:

```cs title="<a href='https://github.com/OpenVidu/openvidu-livekit-tutorials/blob/master/application-server/dotnet/Program.cs#L54-L74' target='_blank'>Program.cs</a>" linenums="54"
string CreateLiveKitJWT(string roomName, string participantName)
{
    JwtHeader headers = new(new SigningCredentials(new SymmetricSecurityKey(Encoding.UTF8.GetBytes(LIVEKIT_API_SECRET)), "HS256")); // (1)!

    var videoGrants = new Dictionary<string, object>() // (2)!
    {
        { "room", roomName },
        { "roomJoin", true }
    };
    JwtPayload payload = new()
    {
        { "exp", new DateTimeOffset(DateTime.UtcNow.AddHours(6)).ToUnixTimeSeconds() }, // (3)!
        { "iss", LIVEKIT_API_KEY }, // (4)!
        { "nbf", 0 }, // (5)!
        { "sub", participantName }, // (6)!
        { "name", participantName },
        { "video", videoGrants }
    };
    JwtSecurityToken token = new(headers, payload);
    return new JwtSecurityTokenHandler().WriteToken(token); // (7)!
}
```

1. Create a new `JwtHeader` with `LIVEKIT_API_SECRET` as the secret key and HS256 as the encryption algorithm.
2. Create a new Dictionary with the video grants for the participant. `roomJoin` allows the user to join a room and `room` determines the specific room. Check out all [Video Grants](https://docs.livekit.io/realtime/concepts/authentication/#Video-grant){:target="\_blank"}.
3. Set the expiration time of the token. LiveKit default's is 6 hours.
4. Set the API key of LiveKit Server as the issuer (`iss`) of the token.
5. The `Not before` field (`nbf`) sets when the token becomes valid (0 for immediately valid).
6. Set the participant's name in the claims `sub` and `name`.
7. Finally, the returned token is sent back to the client.

This method uses the native `System.IdentityModel.Tokens.Jwt` library to create a JWT token valid for LiveKit. The most important field in the JwtPayload is `video`, which will determine the [VideoGrant](https://docs.livekit.io/realtime/concepts/authentication/#Video-grant){:target="\_blank"} permissions of the participant in the Room. You can also customize the expiration time of the token by changing the `exp` field, and add a `metadata` field for the participant. Check out all the available [claims](https://docs.livekit.io/realtime/concepts/authentication/#Token-example){:target="\_blank"}.

Finally, the returned token is sent back to the client.
