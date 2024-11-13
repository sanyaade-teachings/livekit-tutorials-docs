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

## Sync changes between _openvidu.io_ and _livekit-tutorials.openvidu.io_

Whenever any changes are made to the tutorials documentation, theses changes must be also reflected in repository [openvidu.io](https://github.com/OpenVidu/openvidu.io) so they end up available in [openvidu.io/docs/getting-started](https://openvidu.io/docs/getting-started/).

To apply changes in the web _openvidu.io_:

- In this repository, push the changes to tutorials documentation to the `main` branch and run GitHub Action [Publish Web](https://github.com/OpenVidu/livekit-tutorials-docs/actions/workflows/publish-web.yaml) selecting the `main` branch.
- In repository [openvidu.io](https://github.com/OpenVidu/openvidu.io), push the changes to the `main` branch and run GitHub Action to [overwrite the latest version](https://github.com/OpenVidu/openvidu.io#overwriting-the-latest-version).
