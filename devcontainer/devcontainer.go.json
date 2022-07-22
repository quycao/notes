{
    "name": "highhouse_api",           // A free to define name
    // "dockerFile": "dockerfile_debian",   // The path to the Dockerfile which defines the Docker image. It is also possible to directly use an existing Docker image or a Docker Compose file
    "image": "golangdev:1.18.2-alpine3.15",
    "runArgs": [
        "--name=highhouse_api"
        // "--network=highhouse_network",
	    // "--privileged"
    ],
    "settings": {                           // Settings which should be applied to Visual Studio Code. Here the default terminal is set to bash
        "terminal.integrated.defaultProfile.linux": "sh"
    },
    "remoteEnv": {
        "PATH": "/go/bin:/usr/local/go/bin:/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin",
        "GOPATH": "/go"
    },
    "extensions": [                         // Extensions which should be installed in Visual Studio Code by default
        "golang.Go"
    ],
    "appPort": [                            // Mapping host port : container port
        "8000:8000"
    ],
    // "forwardPorts": [                       // The ports which should be forwarded from the container to your localhost, to be able to access, e.g. web pages. Here port 8080 is forward on which the UI5 tooling serves the UI5 application. Port 35729 is forwarded too, for the live reload functionality.
    //     8080,
    //     5000
    // ],
    // "remoteUser": "node"                    // The user which is used to connect to the remote container, if not root should be used. node is a predefined user by the image I used as base
}
