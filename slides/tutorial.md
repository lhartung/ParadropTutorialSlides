class: center, middle
name: title

# Edge Computing with ParaDrop

### Lance Hartung
### Varun Chandrasekaran
### Suman Banerjee

---

class: text-only
name: agenda

# Agenda

1. Technical introduction
  - Edge computing
  - ParaDrop design and implementation
  - Cloud controller and other developer tools
2. Guided setup
  - Account setup and accessing a ParaDrop node
3. Self-directed exercises

???

- Also have a break in the middle.
- Take your time with the exercises, get the most out of them.
- Extra time, try implementing your own ideas.

---

name: edge-computing

.left-column[
# Edge Computing
]

.right-column[
- Move computation closer to users
- Reduce the effect of network latency
- Builds off the cloud computing paradigm

![Default-aligned image](figures/edge-computing.svg)
]

???

- Suman's talk will probably cover motivation for edge computing.
- Key point is that we use same techniques and technologies as cloud computing.

---

.left-column[
# Edge Computing
## Content Delivery Networks
]

.right-column[
* Early predecessor to edge computing
* Focus entirely on caching static content
* No programmability at the edge

![Default-aligned image](figures/content-delivery-network.svg)
]

???

- Akamai launched its CDN in the 1990s.
- Cache static content, especially images and videos.
- The key insight was there - latency matters.
- But all the processing and dynamic behavior still happens in the cloud server.

---

.left-column[
# Edge Computing
## Content Delivery Networks
## Mobile Edge Computing
]

.right-column[
* Address latency inherent to 3G/4G packet processing
* Important aspect of 5G technology development
* Standards in development but unlikely to be completely open

![Default-aligned image](figures/mobile-edge-computing.svg)
]

???

- Important technology that should be on our radar as edge computing researchers.
- Current generation cellular send packets potentially far away through the core.
  - Implement mobility, accounting.
  - However, this adds latency.
- Feature to offload traffic to local servers is gradually being developed and standardized.
- Will likely follow a IaaS/PaaS type model like AWS but remains to be seen.

---

.left-column[
# Edge Computing
## Content Delivery Networks
## Mobile Edge Computing
## ParaDrop
]

.right-column[
* Focus on the "extreme" edge
* Includes Wi-Fi routers in homes and offices
* Unique vantage point and developer opportunities
* Based on an open source software stack

![Default-aligned image](figures/paradrop-edge-computing.svg)
]

???

- Finally, take a look at edge computing in the far reaches of the network.
- Paradrop fills this niche by modifying the WiFi access point to make it programmable.
- Unique opportunities based on types of devices that connect here.
- Also challenging due to the coordination of distributed nodes and different ownership models.

---

name: extreme-edge

# More about the "Extreme" Edge

* Wireless gateway is the hub for connected devices (IoT)
* Access to the wireless medium (Wi-Fi, Bluetooth, etc.) &rarr; unique opportunities
* Always-on, low latency, privacy-preserving programmable substrate

![Default-aligned image](figures/extreme-edge.svg)

???

- Let's talk a little more about the extreme edge and some inspiring use cases.
- Suman probably mentioned the smart home and IoT.
- Can use edge computing to perform computation in a way that respects privacy.
- Also want to broaden our scope to look at smart cities with connected vehicles.
  - Could be drones used by emergency responders.
  - Coordination of traffic at road intersections.
  - One of the exercises you will do today will analyze video from a traffic camera.

---

name: design
class: text-only

.left-column[
# ParaDrop Design
]

.right-column[
* Open source platform built on Linux
* Ubuntu Core
  * Very lightweight distribution (<500 MiB)
  * Transactional updates and signed packages
* Docker for containerization
  * Isolation, portable packaging, reusable images
* Python-based ParaDrop agent
  * Manage applications and system configuration
* Add-on modules using Ubuntu Core's snap packaging
  * Examples: voice recognition, virtual camera
  * Optional depending on hardware and desired functionality
]

---

.left-column[
# ParaDrop Design
## Block Diagram
]

.right-column[
![Default-aligned image](figures/paradrop-block-diagram.svg)
]

???

- This figure shows the layers of software and hardware.
- The applications at top are also referred to as "chutes".
  - Chutes can use APIs exposed by ParaDrop or our add-on modules.
- The ParaDrop daemon communicates with Docker to manage the applications.
- Each of these middle modules is installed as a snap.
  - That means the version is carefully controlled by the operating system.
- Finally, the hardware substrate could be a WiFi router, a server, a Raspberry Pi.
  - It could be x86 or ARM, and so on.

---

.left-column[
# ParaDrop Design
## Block Diagram
## Docker
]

.right-column[
- Built on Linux containers with a friendly interface
- Software isolation with lower overhead than VMs
- Widespread adoption in industry and cloud computing
- Many well-maintained images available in Docker Hub
  - Examples: MySQL, Redis, MongoDB, Tomcat
- Dockerfile is a portable recipe for building an application image

```Dockerfile
FROM node:8

# Set application working directory in the image filesystem.
WORKDIR /opt/app

# Install dependencies listed in package.json using npm.
COPY package.json /opt/app/
RUN npm install

# Copy application source code into the image.
COPY . /opt/app/

EXPOSE 3000
CMD ["node", "index.js"]
```
]

???

- This is an example of a Dockerfile for a service built using node.js.
  - It inherits from the Docker image maintained by node.js developers.
  - It installs some language-level packages through npm.
  - It installs its own application code.
  - Then it configures the exposed network port and command to start it.
- Any machine running Docker can install this service in a reproducible way.

---

class: text-only

.left-column[
# ParaDrop Design
## Block Diagram
## Docker
## Add-ons
]

.right-column[
- Extend the capabilities of the platform
  - Depending on specific hardware and applications
- **Airshark**
  - Detect and classify non-Wi-Fi interferance
  - E.g. microwave ovens and Bluetooth
  - Requires specific Atheros hardware
- **Voice**
  - Play an audio prompt and perform voice recognition
  - Requires speaker and microphone
- **Image Server**
  - Serve images as a virtual camera
  - Used for this tutorial in place of physical cameras
]

???

- Airshark was a research project at UW-Madison.
- Voice module could be used to implement functions like a smart speaker / voice assistant.
- Image server is a web server that provides a different frame for each request.
- These are a few examples of add-ons we have implemented.

---

name: internals
class: text-only

.left-column[
# ParaDrop Internals
]

.right-column[
- Agent runs as a daemon on the edge compute node
- Manage and supervise applications
  - Build and install Docker containers
  - Facilitate application access to hardware
  - Networking between containers, LAN, and WAN
- Expose edge API (HTTP and websocket)
  - Enforce user access permissions
- Maintain contact with controller (HTTP and websocket)
  - Apply changes scheduled through controller
  - Send status information and log messages
]

---

.left-column[
# ParaDrop Internals
## Block Diagram
]

.right-column[
![Default-aligned image](figures/paradrop-internal-design.svg)
]

---

.left-column[
# ParaDrop Internals
## Block Diagram
## Execution Pipeline
]

.right-column[
- System changing operations follow a pipeline
- Each stage has an associated rollback function
- Validate, gather information, then commit to change
- Some concurrency &mdash; prepare images stage

![Default-aligned image](figures/execution-pipeline.svg)
]

???

The goal is to break a complex task such as installing a chute into
smaller tasks while still atomically succeeding or failing.

---

name: controller

.left-column[
# Cloud
## Controller
]

.right-column[
- Web-based interface for remotely managing nodes
- Public chute store with dedicated git repositories
- Cloud API for third-party integration
- **Not** on the data path for applications

![Default-aligned image](figures/paradrop-cloud-controller.svg)
]

---

.left-column[
# Cloud
## Controller
## API
]

.right-column[
## Cloud API
- Cloud API empowers 3rd party applications
- Make use of node management and *paradrop* function
- Controller enforces ownership and permissions

```bash
curl -H "Authorization: Bearer eyJhbGciOiJIUzI1NiIsIn..." \
https://paradrop.org/api/routers
```
```json
[{
  "_id": "59f21df1081e8678f8967276",
  "updatedAt": "2018-10-09T16:15:09.775Z",
  "createdAt": "2017-10-26T17:40:01.150Z",
  "name": "black01",
  "platform_version": "0.11.2",
  "number_chutes": 3,
  "online": true,
  ...
}]
```
]

---

name: development
class: text-only

.left-column[
# Develop
## Tools
]

.right-column[
## Tools for development and testing
- Node administration panel
  - Access real-time status, logs, and chute output
- Cloud controller (paradrop.org)
  - Manage multiple nodes
  - Install chutes from the chute store
- Command line tool (pdtools)
  - Many functions for directly controlling nodes
- SSH
  - Shell access to computing environment and Docker
]

---

.left-column[
# Develop
## Tools
## Chute Config
]

.right-column[
**Example paradrop.yaml file**

```yaml
name: traffic-camera # Basic information about the chute.
version: 1
description: |
  Use an OpenCV cascade classifier to detect and count
  vehicles in images from a camera mounted on a street pole.
environment:         # Services will inherit these variables:
  IMAGE_SOURCE_URL: "http://paradrop.io:7466/park/video.jpg"
  CASCADE_FILE: "vehicle.xml"
  MASK_FILE: "masks/park.png"
services:            # Chutes can have multiple services, but
  main:              # all chutes should have a main service.
    command: python2 -u -m chute
    image: python2-cv
    source: .
    type: light
web:                 # This chute runs a web server
  port: 5000         # listening on port 5000
  service: main      # as part of the main service.
```

If this looks similar to a Docker Compose file, that is not by coincidence.
]

---

name: logistics

.left-column[
# Tutorial
## Setup
]

.right-column[
- Physical nodes (with Wi-Fi capabilities) in our U.S. lab
- Virtual compute nodes in EC2 Mumbai datacenter
- Laptops with tutorial VM and developer tools

![Default-aligned image](figures/tutorial-setup.svg)
]

---

class: text-only

.left-column[
# Tutorial
## Setup
## Logistics
]

.right-column[
1. Make sure your laptop has the tutorial VM running.
  - https://github.com/ParadropLabs/ParadropTutorial
2. Visit paradrop.org and claim a compute node.
  - We will give you a handout with the claim token.
  - We may assign 2-3 people to each node.
3. Introduce the various developer tools together.
  - paradrop.org and pdtools
4. Follow the self-guided tutorial in the handout.
  - There is a link for the Google Doc on the GitHub page.
]
