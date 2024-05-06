# livekit-tutorials-docs

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
