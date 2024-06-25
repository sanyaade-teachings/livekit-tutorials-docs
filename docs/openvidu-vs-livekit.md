# Running OpenVidu vs LiveKit locally

If you want to run the tutorials on this site locally without the need to create an account on [LiveKit Cloud](https://cloud.livekit.io/){target="_blank"}, we recommend **running OpenVidu locally**. OpenVidu is 100% compatible with LiveKit, and it offers many **advantages** for local development:

- Egress and Ingress services already integrated with a Redis instance. [See more](#egress-and-ingress-out-of-the-box).
- S3 compatible storage for Egress recordings. [See more](#s3-compatible-storage).
- Administration dashboard to monitor your Rooms. [See more](#administration-dashboard).
- OpenVidu Call: a ready-to-use app. [See more](#openvidu-call).
- Access your application from any device in your local network. [See more](#accessing-your-app-from-other-devices-in-your-network).

Whenever you are ready to deploy your real-time application, OpenVidu for production brings awesome features. Visit [What is OpenVidu?](./about-openvidu.md) to learn more.

=== "Run OpenVidu locally"

	--8<-- "docs/tutorials/shared/run-openvidu-locally.md"

### Egress and Ingress out of the box

LiveKit allows you to export media from a Room (for example recording it) or import media into a Room (for example ingesting a video file), using [Egress](https://docs.livekit.io/realtime/egress/overview/){target="_blank"} and [Ingress](https://docs.livekit.io/realtime/ingress/overview/){target="_blank"} services respectively. These modules are independent of LiveKit Server and must be correctly configured and connected via a shared Redis.

When running OpenVidu locally you will have all these services properly integrated, so you can focus on developing your app without worrying about anything else.

### S3 compatible storage

If you are planning to make use of the [Egress](https://docs.livekit.io/realtime/egress/overview/){target="_blank"} service to export media out of your Rooms, you will need an S3 compatible bucket to store the generated files, and configure your LiveKit deployment to use it.

When running OpenVidu locally you will have an S3 compatible storage available right away ([Minio](https://min.io/){target="_blank"}).

### Administration dashboard

OpenVidu comes with an administration dashboard that allows you to monitor the status of your Rooms. Not only in real time, but also historically: the number of participants, the number of published tracks, Egress and Ingress processes... This is a great tool to have when developing your app, as it can help spotting issues and debugging your application's logic.

### OpenVidu Call

When running OpenVidu locally a default application will be available for you to try. We have named this application **OpenVidu Call**, and it gathers the most important features that advanced videonconference apps would require: camera selection, screen sharing, virtual backgrounds, chat, recording... 

You can use OpenVidu Call to help develop your own application, joining participants to shared rooms between your app and OpenVidu Call, and seeing how your app behaves.

### Accessing your app from other devices in your network

Testing WebRTC applications can be tricky, as devices require a secure context (HTTPS) to allow accessing the camera and microphone. This is not a problem when accessing your app from the same computer where LiveKit Server is running, as `localhost` is considered a secure context (even if it is served over plain HTTP). So, given this architecture:

<figure markdown="span">
  ![Image title](./assets/images/livekit-architecture.svg){ width="450" style="border-radius: 8px" }
</figure>

The most straightforward way to test your app is:

- Run LiveKit Server on your computer.
- Connect your app to LiveKit Server through plain `localhost`.
- Serve your application client and application server from the same computer.
- Access your app from `localhost` in a browser of the same computer.
  
This is fairly simple, but what if you want to test your app from different devices at the same time? And what if you want to test it from a real mobile device? In this case there is no other option than using a secure context (HTTPS). And this brings two different problems:

1. LiveKit Server does not provide HTTPS support out of the box. A reverse proxy is needed to serve LiveKit Server over HTTPS.
2. Even when serving LiveKit Server and accessing your app from an HTTPS address of your local network, the SSL certificate won't just be valid. You must accept it in the browser for web apps, and you will have to install it in your mobile devices for mobile apps.

To overcome these problems, you can [run OpenVidu locally](#run-openvidu-locally). It comes with a magic domain name `openvidu-local.dev`{.do-not-break-line} which can resolve to any IP specified as a subdomain, and it offers a valid wildcard certificate for HTTPS (this is similar to [nip.io](https://nip.io), [traefik.me](https://traefik.me) or [localtls](https://github.com/Corollarium/localtls)).

The result is that you can access your application client through `https://xxx-yyy-zzz-www.openvidu-local.dev:5443`{.do-not-break-line} where **`xxx-yyy-zzz-www`** is your LAN private IP address with dashes (-) instead of dots (.). The SSL certificate will be valid!

!!! info

    For example, if your local IP address is **`192.168.1.52`** you can access your app from any device in your network through **`https://192-168-1-52.openvidu-local.dev:5443`**{.do-not-break-line}

!!! warning

    This OpenVidu deployment is only intended for local development and testing. It is not ready for production environments in terms of networking and security. For production deployments, you should follow the [official OpenVidu installation guide](https://openvidu.io/docs/self-hosting/deployment-types/){target="_blank"}.

<br>