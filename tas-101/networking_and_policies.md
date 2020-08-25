---
title : "Networking & Policies (Advanced)"
weight: 60
---

# Networking & Policies (Advanced)
## Target the Environment

This example demonstrates communication (HTTP and UDP) between a frontend and backend applications using service discovery over the container network.

1) Follow the earlier instructions to Get Environment Access

1) Clone or download the repo into a path [CF Networking Examples](https://github.com/nicksterling/cf-networking-examples) :

## Frontend

The frontend allows you to test out container network communication via two methods:

- Connect to the backend via HTTP
- Connect to the backend via UDP

In either case, the response from the backend to the frontend will be rendered as a web page.

### Deploying

        $ cd frontend
        $ cf push frontend

**Use Case: Frontend Connects to Backend**

## Backend

The backend will be pushed with no external route and therefore should not be accessible via the public internet.

Backend serves a picture of a typing cat on the TCP ports specified in the environment variable CATS_PORTS, and responds to simple text messages on the UDP ports specified in the environment variable UDP_PORTS.

We will give the backend an internal hostname that will map to the app’s container ips and can be used to connect via container-to-container networking. An internal hostname is configured via the CF CLI map-route command, with the domain provided set to the reserved internal domain of apps.internal.
### Deploying

Backend (example with internal route defined in app manifest)

        $ cd $DIR/backend
        $ cf push backend --no-start
        $ cf set-env backend CATS_PORTS "7007,7008"
        $ cf set-env backend UDP_PORTS "9003,9004"
        $ cf start backend

Usage

After both frontend and backend apps have been deployed, you can visit the frontend URL in a browser. You should see something like:

        Frontend Sample App

        HTTP Test
        Backend HTTP URL: [....] [ Submit ]

        UDP Test
        Backend UDP Server Address: [....]
        Message: [....] [ Submit ]

## Usage with HTTP

In the frontend app, in the Backend HTTP URL field enter backend’s internal hostname and a cats port (_backend.apps.internal:7007_). Hit submit.

You will see an error message saying the connection is refused. This is because the two apps have not been configured to allow connections from the frontend to the backend.

If you see an error message saying no such host, service discovery is incorrectly configured.

Now allow access:

        cf add-network-policy frontend --destination-app backend --port 7007 --protocol tcp

Now if you try again from the frontend:

    [GIF OF CAT]
    Hello from the backend, port: 7007

Doing the same thing with backend-b should result in a different cat (specifically, a party cat) being shown.

## Usage with UDP

In the frontend app, in the Backend UDP Server Address field, enter the backend’s internal hostname and UDP port (_backend.apps.internal:9003_) and a message. Hit submit.

You will see an error message. This is because the two apps have not been configured to allow connections from the frontend to the backend.

Now allow access:

        cf add-network-policy frontend --destination-app backend --port 9003 --protocol udp

Now if you try again from the frontend:

        You sent the message: hello world

        Backend UDP server replied: HELLO WORLD
