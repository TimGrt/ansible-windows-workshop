# Ansible Workshop Exercises

This repository contains all exercise descriptions for the Ansible Windows Workshop.  
The exercises are build with MkDocs and published to [Github pages](https://timgrt.github.io/ansible-windows-workshop/).  
A Dockerfile is included to build a container image which publishes the exercises as a Webserver.

[![Linting](https://github.com/TimGrt/ansible-windows-workshop/actions/workflows/ci.yml/badge.svg)](https://github.com/TimGrt/ansible-windows-workshop/actions/workflows/ci.yml)

## Development

> Use the provided `Makefile` to setup the development environment!

The `all` target creates a Python VE with all requirements and installs the provided *pre-commit* hooks:

```console
make all
```

Running without specifying a target displays a *help* message.

Use the `clean` target to remove the development environment again after you are finished.

### Manual setup

Create a Python virtual environment:

```bash
python3 -m venv ve-mkdocs-dev
```

Activate VE:

```bash
source ve-mkdocs-dev/bin/activate
```

Install requirements from project directory:

```bash
pip3 install -r requirements.txt
```

Install *pre-commit* package and hooks:

```bash
pip3 install -r requirements.txt
```

```bash
pre-commit install
```

You can run all hooks at any time:

```bash
pre-commit run -a
```

Get IP address:

```bash
hostname -I
```

Start MkDocs built-in dev-server for live-preview:

```bash
mkdocs serve -a 172.26.220.226:8080
```

## Pull Request Guidelines

Before opening a *pull request* make sure you followed the next couple of steps.

1. Use the provided `Makefile` to create a development environment!
2. Always **preview** the changes you made thoroughly, only commit your changes if everything looks as intended!
3. Use the provided *pre-commit* configuration, it will lint your Markdown files and also check for spelling errors!

## Build and publish guide manually

Use the provided `Containerfile` if you want to publish the Workshop exercises guide yourself.

Build the image:

```bash
podman build -t ansible-windows-workshop .
```

Run a container from the previously build image, the webserver is available at Port 8080:

```bash
podman run -d -p 8080:8080/tcp --name workshop ansible-windows-workshop
```
