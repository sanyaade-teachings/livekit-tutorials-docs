## Intro

[OpenVidu](https://openvidu.io){target=\_blank} is a supercharged, LiveKit-based, real-time solution. With it you can add real-time audio and video capabilities to your application: build any kind of videoconference experience, serve ultra-low latency livestreams to thousands of users, manage real-time videos from your drones or camera feeds and record them... The possibilities are truly endless!

OpenVidu takes the great foundation that is the open source version of LiveKit and takes it to the next level in the following areas:

- :material-server: **On-Premises**: take full control of your resources and data. OpenVidu is designed to be easily be deployed in your own infrastructure, keeping all of your application's data completely secure without ever leaving your domain. The management, maintenance and updating of your cluster becomes an easy task with OpenVidu. We also provide ready-to-use native deployments in the most popular cloud providers.
- :material-lightning-bolt: **Performant**: OpenVidu is built to be incredibly powerful, providing up to 2x performance of LiveKit. You can host double the users in your video rooms just by using OpenVidu.
- :material-shield-refresh: **Fault tolerance**: OpenVidu deployments are natively fault tolerant in all of their components. In real-time applications, downtime events are extremely disruptive, so with OpenVidu node failures are handled appropriately to provide a seamless experience with no downtime.
- :material-chart-timeline-variant-shimmer: **Scalable**: add nodes to increase the capacity of your cluster when necessary and remove them when no longer needed to optimize your resources and squeeze the most out of them.
- :material-microscope: **Observable**: OpenVidu provides different tools and APIs to monitor the status, health, performance and history of your deployment. It automatically collects logs and metrics and offers a dashboard to visualize them.

<figure markdown>
  ![OpenVidu vs LiveKit](../assets/images/openvidu-vs-livekit.png){ .mkdocs-img }
  <figcaption>OpenVidu is a 100%-compatible fork of LiveKit and extends it with custom addons and APIs. These extensions provide improved performance, new features and facilitate the deployment and management of your cluster.</figcaption>
</figure>

## Developing your app with OpenVidu vs LiveKit

All of the tutorials on this site are ready to be used against a LiveKit Server, whether you are [running it locally](https://docs.livekit.io/home/self-hosting/local/){target="_blank"} or using [LiveKit Cloud](https://cloud.livekit.io/){target="_blank"}.

But we recommend **running OpenVidu locally**, which will get you the following **benefits**:

- Egress and Ingress services already integrated with a Redis instance. [See more](#egress-and-ingress-out-of-the-box).
- S3 compatible storage for Egress recordings. [See more](#s3-compatible-storage).
- Administration dashboard to monitor your Rooms. [See more](#administration-dashboard).
- OpenVidu Call: a ready-to-use app. [See more](#openvidu-call).
- Access your application from any device in your local network. [See more](#accessing-your-app-from-other-devices-in-your-network).

=== "Run OpenVidu locally"

	--8<-- "docs/tutorials/shared/run-openvidu-locally.md"

### Egress and Ingress out of the box

LiveKit allows you to export media from a Room (for example recording it) or import media into a Room (for example ingesting a video file), using [Egress](https://docs.livekit.io/realtime/egress/overview/){target="_blank"} and [Ingress](https://docs.livekit.io/realtime/ingress/overview/){target="_blank"} services respectively. These modules are independent of LiveKit Server and must be correctly configured and connected via a shared Redis.

When running OpenVidu locally you will have all these services properly integrated, so you can focus on developing your app without worrying about anything else.

### S3 compatible storage

If your planning to make use of the [Egress](https://docs.livekit.io/realtime/egress/overview/){target="_blank"} service to export media out of your Rooms, you will need an S3 compatible bucket to store the generated files, and configure your LiveKit deployment to use it.

When running OpenVidu locally you will have an S3 compatible storage available right away ([Minio](https://min.io/){target="_blank"}).

### Administration dashboard

OpenVidu comes with an administration dashboard that allows you to monitor the status of your Rooms. Not only in real time, but also historically: the number of participants, the number of published tracks, Egress and Ingress processes... This is a great tool to have when developing your app, as it can help spotting issues and debugging your application's logic.

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

    This OpenVidu deployment is only intended for local development and testing. It is not ready for production environments in terms of networking and security. For production deployments, you should follow the [official OpenVidu installation guide](https://openvidu.io/installation/){target="_blank"}.

### OpenVidu Call

When running OpenVidu locally a default application will be available for you to try. We have called this application **OpenVidu Call**, and it gathers the most important features that advanced videonconference apps would require: camera selection, screen sharing, virtual backgrounds, chat, recording... 

You can use OpenVidu Call to help develop your own application, joining participants to shared rooms between your app and OpenVidu Call, and seeing how your app behaves.

## OpenVidu editions

OpenVidu is available in two editions:

- **OpenVidu** <a href="/pricing#openvidu-community"><span class="openvidu-tag openvidu-community-tag">COMMUNITY</span></a>: free to use. It is a single-server deployment and provides a custom LiveKit distribution with Egress, Ingress, S3 storage and monitoring. Ideal for development, testing and small-scale production deployments.
- **OpenVidu** <a href="/pricing#openvidu-pro"><span class="openvidu-tag openvidu-pro-tag">PRO</span></a>: OpenVidu commercial edition. It is a multi-server deployment with all the features of OpenVidu Community plus 2x performance, scalability, fault tolerance and improved monitoring and observability. Ideal for large-scale production deployments that require the highest standards.

Visit [openvidu.io](https://openvidu.io) to learn more about what OpenVidu has to offer.

<br>
