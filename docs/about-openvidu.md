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

But we recommend running OpenVidu locally, which will get you the following benefits:

- [Egress](https://docs.livekit.io/realtime/egress/overview/){target="_blank"} and [Ingress](https://docs.livekit.io/realtime/ingress/overview/){target="_blank"} services already integrated with a Redis instance.
- S3 compatible storage for Egress recordings.
- Access your application from any device in your local network (see [more information](#accessing-your-app-from-other-devices-in-your-network)).

=== "Run OpenVidu locally"

	--8<-- "docs/tutorials/shared/run-openvidu-locally.md"

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

To overcome these problems, you can [run OpenVidu locally](#run-openvidu-locally). It comes with a magic domain name `openvidu-local.dev` which can resolve to any IP specified as a subdomain, and it offers a valid wildcard certificate for HTTPS (this is similar to [nip.io](https://nip.io), [traefik.me](https://traefik.me) or [localtls](https://github.com/Corollarium/localtls)).

The result is that you can access your application client through `https://xxx-yyy-zzz-www.openvidu-local.dev:5443`{.do-not-break-line}, where **xxx-yyy-zzz-www** is your LAN private IP address with dashes (-) instead of dots (.). The SSL certificate will be valid!

!!! info

    For example, if your local IP address is **`192.168.1.52`** you can access your app from any device in your network through **`https://192-168-1-52.openvidu-local.dev:5443`**{.do-not-break-line}

## OpenVidu editions

OpenVidu is available in two editions:

- **OpenVidu** <a href="/pricing#openvidu-community"><span class="openvidu-tag openvidu-community-tag">COMMUNITY</span></a>: 
- **OpenVidu** <a href="/pricing#openvidu-pro"><span class="openvidu-tag openvidu-pro-tag">PRO</span></a>:
