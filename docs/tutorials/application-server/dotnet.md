# openvidu-basic-dotnet

[Source code :simple-github:](https://github.com/OpenVidu/openvidu-livekit-tutorials){ .md-button target=\_blank }

This is a minimal server application sample built for .NET with [ASP.NET Core Minimal APIs](https://docs.microsoft.com/aspnet/core/tutorials/min-web-api?view=aspnetcore-6.0&tabs=visual-studio){:target="\_blank"}.
It internally builds JWT tokens for LiveKit.

## Running this application

#### Prerequisites

To run this application you will need **.NET**:

- [.NET 8.0](https://dotnet.microsoft.com/en-us/download){:target="\_blank"}

#### Download repository

```bash
git clone https://github.com/OpenVidu/openvidu-livekit-tutorials.git
cd openvidu-livekit-tutorials/openvidu-basic-dotnet
```

#### Run application

```bash
dotnet run
```

## Understanding the code

The application is a simple ASP.NET Core Minimal APIs app with a single file `Program.cs` that exports a unique endpoint:

- `/token` : Generate a token for a given Room name and Participant name

<!-- > You can get more information about these endpoints in the [Application Server Endpoints](application-server/#rest-endpoints) section. -->

Let's see the code of the `Program.cs` file:

```cs
var builder = WebApplication.CreateBuilder(args);
var MyAllowSpecificOrigins = "_myAllowSpecificOrigins";

IConfiguration config = new ConfigurationBuilder()
                .SetBasePath(Directory.GetCurrentDirectory())
                .AddJsonFile("appsettings.json")
                .AddEnvironmentVariables().Build();

// Load env variables
var SERVER_PORT = config.GetValue<int>("SERVER_PORT");
var LIVEKIT_API_KEY = config.GetValue<string>("LIVEKIT_API_KEY");
var LIVEKIT_API_SECRET = config.GetValue<string>("LIVEKIT_API_SECRET");

// Enable CORS support
builder.Services.AddCors(options =>
{
    options.AddPolicy(name: MyAllowSpecificOrigins,
                      builder =>
                      {
                          builder.WithOrigins("*").AllowAnyHeader();
                      });
});

builder.WebHost.UseKestrel(serverOptions =>
{
    serverOptions.ListenAnyIP(SERVER_PORT);
});

var app = builder.Build();
app.UseCors(MyAllowSpecificOrigins);
```

Starting by the top, the `Program.cs` file has the following fields:

- `builder`: a `WebApplicationBuilder` instance to build the application.
- `MyAllowSpecificOrigins`: the name of the CORS policy to be used in the application.
- `config`: a `IConfiguration` instance to load the configuration from the `appsettings.json` file.
- `SERVER_PORT`: the port where the application will be listening.
- `LIVEKIT_API_KEY`: the URL of your OpenVidu deployment.
- `LIVEKIT_API_SECRET`: the secret of your OpenVidu deployment.
- `app`: the `WebApplication` instance.

The first thing the application does is to configure CORS support. The CORS policy is configured to allow requests from any origin and any header.

#### Create token endpoint

The unique endpoint of the application is `/token`. It receives a JSON object with the following fields:

- `roomName`: the name of the Room where the user wants to connect.
- `participantName`: the name of the participant that wants to connect to the Room.

```csharp
app.MapPost("/token", async (HttpRequest request) =>
{
    var body = new StreamReader(request.Body);
    string postData = await body.ReadToEndAsync();
    Dictionary<string, dynamic> bodyParams = JsonSerializer.Deserialize<Dictionary<string, dynamic>>(postData) ?? new Dictionary<string, dynamic>();

    if (bodyParams.TryGetValue("roomName", out var roomName) && bodyParams.TryGetValue("participantName", out var participantName))
    {
        var token = CreateLiveKitJWT(roomName.ToString(), participantName.ToString());
        return Results.Json(token);
    }
    else
    {
        return Results.BadRequest("\"roomName\" and \"participantName\" are required");
    }
});
```

The endpoint obtains a Dictionary from the body request, and check if fields `roomName` and `participantName` are available. If not, it returns a `400` error.

If required fields are available, a new JWT token is created. Unfortunately there is no .NET SDK for LiveKit, so we need to create the JWT token manually. The `CreateLiveKitJWT` method is responsible for creating the JWT token:

```csharp
string CreateLiveKitJWT(string roomName, string participantName)
{
    JwtHeader headers = new(new SigningCredentials(new SymmetricSecurityKey(Encoding.UTF8.GetBytes(LIVEKIT_API_SECRET)), "HS256"));

    var videoGrants = new Dictionary<string, object>()
    {
        { "room", roomName },
        { "roomJoin", true }
    };
    JwtPayload payload = new()
    {
        { "exp", new DateTimeOffset(DateTime.UtcNow.AddHours(6)).ToUnixTimeSeconds() },
        { "iss", LIVEKIT_API_KEY },
        { "nbf", 0 },
        { "sub", participantName },
        { "name", participantName },
        { "video", videoGrants }
    };
    JwtSecurityToken token = new(headers, payload);
    return new JwtSecurityTokenHandler().WriteToken(token);
}
```

This method uses the native `System.IdentityModel.Tokens.Jwt` library to create a JWT token valid for LiveKit. The most important field in the JwtPayload is `video`, which will determine the [VideoGrant](https://docs.livekit.io/realtime/concepts/authentication/#Video-grant){:target="\_blank"} permissions of the participant in the Room. You can also customize the expiration time of the token by changing the `exp` field.

Finally, the returned token is sent back to the client.