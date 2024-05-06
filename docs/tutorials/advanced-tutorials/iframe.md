# openvidu-iframe

[Source code :simple-github:](https://github.com/OpenVidu/openvidu-tutorials){ .md-button target=\_blank }

This tutorial shows how to use an iframe in Livekit applications. It is based on the [openvidu-js](../application-client/javascript.md) tutorial, so we recommend you to read it before this one.

## Running this tutorial

Running this tutorial is straightforward, and here's what you'll need:

<h3>1. OpenVidu Server Installation</h3>

--8<-- "docs/tutorials/shared/run-openvidu-dev.md"

<h3>2. Start your preferred server application sample</h3>

--8<-- "docs/tutorials/shared/application-server-tabs.md"

<h3>3. Launch the client application tutorial</h3>

In order to run the client application tutorial, it's essential to have an HTTP web server installed on your development computer. If you have Node.js installed, you can easily set up [http-server](https://github.com/indexzero/http-server){:target="\_blank"}. Here's how to install it:

```bash
npm install --location=global http-server
```

After successfully installing http-server, you'll need to serve two different applications:

<h4>1. The client application:</h4>

This application will be embedded within an iframe. We'll be using [openvidu-js](../application-client/javascript.md) application for this purpose.

To serve the client application, execute the following command:

```bash
# Using the same repository openvidu-livekit-tutorials from step 2
http-server openvidu-tutorials/openvidu-js/web
```

It will be served in the port **8080**.

<h4>2. The iframe application:</h4>

To serve the iframe application, execute the following command:

```bash
# Using the same repository openvidu-livekit-tutorials from step 2
http-server openvidu-tutorials/openvidu-iframe/web
```

Once the server is up and running, you can test the application by visiting [`http://localhost:8081`](http://localhost:8081){:target="\_blank"}. You should see a screen like this:

<div class="grid-container">

<div class="grid-50"><p><a class="glightbox" href="../../../../assets/images/openvidu-iframe.png" data-type="image" data-width="100%" data-height="auto" data-desc-position="bottom"><img src="../../../../assets/images/openvidu-iframe.png" loading="lazy"/></a></p></div>

<div class="grid-50"><p><a class="glightbox" href="../../../../assets/images/openvidu-iframe2.png" data-type="image" data-width="100%" data-height="auto" data-desc-position="bottom"><img src="../../../../assets/images/openvidu-iframe2.png" loading="lazy"/></a></p></div>

</div>

## Understanding the code

Let's clarify what an `iframe` is. An `iframe` is an HTML tag that allows you to embed another HTML document inside the current one. In this case, we will embed the [openvidu-js](../application-client/javascript.md) application inside the _**openvidu-iframe**_ application. Knowing that, the _openvidu-iframe_ application is extremely simple. It has only 2 files:

- `index.html`: This file contains the HTML code of the application. It has a `div` element that will contain the `iframe` element. It also has a button to start the screen sharing.
- `styles.css`: This file contains the CSS code of the application. It only has the styles of the button.
