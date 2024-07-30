# livekit-tutorials-docs

Visit at [https://livekit-tutorials.openvidu.io/](https://livekit-tutorials.openvidu.io/).

Create custom Docker image with necessary extra plugings:

```bash
docker build --pull --no-cache --rm=true -t squidfunk/mkdocs-material .
```

Serve:

```bash
docker run --name=mkdocs --rm -it -p 8000:8000 -v ${PWD}:/docs squidfunk/mkdocs-material
```

Build:

```bash
docker run --rm -it -v ${PWD}:/docs squidfunk/mkdocs-material build
```

## Updating tutorials

Whenever any changes are made to the tutorials documentation, theses changes must be also reflected in the "tutorials" section of the official OpenVidu documentation, which is located in the [openvidu.io repository](https://github.com/OpenVidu/openvidu.io).

In order to publish the changes in the tutorials, follow these steps:

-   In this repository, push the changes to the `main` branch, go to `Actions` and run the workflow named "Publish Web" selecting the `main` branch.
-   In the [openvidu.io repository](https://github.com/OpenVidu/openvidu.io), push the changes to the `main` branch and execute the script [Overwriting the latest version](https://github.com/OpenVidu/openvidu.io#overwriting-the-latest-version) to update the latest version of the documentation.
