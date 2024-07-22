##### From production to a local server

When developing locally pointing to a production deployment and webhooks events are required by your application, you might face issues because OpenVidu cannot access your local server.

To receive webhooks from OpenVidu on your local machine, you need to expose your local server to the internet. This exposure allows OpenVidu to send webhooks directly to your local server.

The following images illustrate the difference between an unreachable local server and a reachable local server:

<div class="grid cards" markdown>

<figure markdown>
  ![Unreachable local server](../../../assets/images/application-servers/webhook-fail.svg){ loading=lazy .svg-img  .mkdocs-img}
  <figcaption>Unreachable local server</figcaption>
</figure>

<figure markdown>
  ![Reachable local server](../../../assets/images/application-servers/webhook-success.svg){ loading=lazy .svg-img  .mkdocs-img}
  <figcaption>Reachable local server</figcaption>
</figure>

</div>

Exposing your local server to the internet is a common practice when developing applications locally. Tools like [Ngrok](https://ngrok.com/){target="\_blank"}, [LocalTunnel](https://localtunnel.github.io/www/){target="\_blank"}, [LocalXpose](https://localxpose.io/){target="\_blank"} and [Zrok](https://zrok.io/){target="\_blank"} can help you achieve this.

These tools provide you with a public URL that forwards requests to your local server. You can use this URL to receive webhooks from OpenVidu. For information on how to add this URL as the webhook URL in the OpenVidu deployment, refer to the following documentation:

=== "OpenVidu Local"

    [Go to reference](https://openvidu.io/latest/docs/self-hosting/local/#configuring-webhooks){target="\_blank"}

=== "OpenVidu Single Node"

    - Change webhooks in an **On-Premises deployment**. [Go to reference](https://openvidu.io/latest/docs/self-hosting/single-node/on-premises/admin/#enabling-webhooks){target="\_blank"}.

    - Change webhooks in an **AWS deployment**. [Go to reference](https://openvidu.io/latest/docs/self-hosting/single-node/on-premises/admin/#enabling-webhooks){target="\_blank"}.


=== "OpenVidu Elastic"

    - Change webhooks in an **On-Premises deployment**. [Go to reference](https://openvidu.io/latest/docs/self-hosting/elastic/on-premises/admin/#enabling-webhooks){target="\_blank"}.

    - Change webhooks in an **AWS deployment**. [Go to reference](https://openvidu.io/latest/docs/self-hosting/elastic/aws/admin/#enabling-webhooks){target="\_blank"}.

=== "OpenVidu High Availability"

    - Change webhooks in an **On-Premises deployment**. [Go to reference](https://openvidu.io/latest/docs/self-hosting/ha/on-premises/admin/#enabling-webhooks){target="\_blank"}.

    - Change webhooks in an **AWS deployment**. [Go to reference](https://openvidu.io/latest/docs/self-hosting/ha/aws/admin/#enabling-webhooks){target="\_blank"}.
