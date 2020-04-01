# VS Code Remote Development

## 1. Recipe
-  Install [VS Code Remote-Containers](https://marketplace.visualstudio.com/items?itemName=ms-vscode-remote.remote-containers) extension.
- Write a Dockerfile defining your image with packages and tools required to run and debug your application.
- Create a .devcontainer in your project and then add Dockerfile and devcontainer.json to it, if you want to use the docker image that is on your local machine already, you can no need the docker file and specify the docker image in the devcontainer.json file.
- Open your project - that containing your .devcontainer folder with VS Code, it will automatically detect it and ask you to reopen the project in container mode.
- Then VS Code will build Docker images and containers based on your Dockerfile (only for first time) and setup some additional required packages so that it can work containerizedly.
- After the process is finished, you are able to use VS Code as normal and have everything actually run inside containers.

## 2. Implementation
- Following the concept and recipe mentioned above, I create a Dockerfile
```docker
FROM golang:alpine

# Install git and build-base (gcc, musl-dev...) for delve debug.
# Git is required for fetching the dependencies.
RUN apk update && apk add --no-cache git build-base

# Install essential tools for Go development.
RUN go get \
        golang.org/x/tools/gopls \
        github.com/mdempsky/gocode \
        github.com/uudashr/gopkgs/cmd/gopkgs \
        github.com/ramya-rao-a/go-outline \
        github.com/acroca/go-symbols \
        golang.org/x/tools/cmd/guru \
        golang.org/x/tools/cmd/gorename \
        github.com/cweill/gotests/... \
        github.com/fatih/gomodifytags \
        github.com/josharian/impl \
        github.com/davidrjenni/reftools/cmd/fillstruct \
        github.com/haya14busa/goplay/cmd/goplay \
        github.com/godoctor/godoctor \
        github.com/go-delve/delve/cmd/dlv \
        github.com/stamblerre/gocode \
        github.com/rogpeppe/godef \
        golang.org/x/tools/cmd/goimports \
        golang.org/x/lint/golint 2>&1 \
    # Clean up.
    && rm -rf $GOPATH/src \
    && rm -rf $GOPATH/pkg
	

# Expose service ports.
EXPOSE 8000
```
- Then I write a devcontainerl.json file
```json
{
    "image": "vscode_go:debian",
    // "dockerFile": "dockerfile_debian",
    "appPort": [
        "8000:8000"
    ],
    "extensions": [
        "ms-vscode.go"
    ]
}

```

## 3. Result
- I was able to write with all of IntelliSense's features, such as autocompletion, auto imports, code navication, etc.
- I was able to run the application without Go runtime installed on my local machine. Thanks to port mapping, I can test my application directly with my Safari, too.
- I was able to use awesome debugging features supported by Go VS Code extension

## 4. Limitations
I pretty much have everything I need. However, there're still some limitations that I feel a bit annoying:
- I have to work around a bit to be able to use VS Code Git Push command, which requires my SSH credentials bound to the containers.
- I could not do GPG sign my commit using VS Code Git Commit command. It's possible but it requires quite a lot of work to be able to forward GPG commands from containers to host machines.


### Reference: https://levelup.gitconnected.com/a-complete-go-development-environment-with-docker-and-vs-code-2355aafe2a96
