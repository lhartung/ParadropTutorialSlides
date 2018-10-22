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

---

name: edge-computing

.left-column[
# Edge Computing
]

.right-column[
- Builds off the cloud computing paradigm
- Move computation closer to users
- Reduce the effect of network latency

![Default-aligned image](figures/edge-computing.svg)
]

---

.left-column[
# Edge Computing
## Content Delivery Networks
]

.right-column[
* Early predecessor to edge computing
* Focus entirely on caching static content
* Low programmability

![Default-aligned image](figures/content-delivery-network.svg)
]

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

---

name: extreme-edge

# More about the "Extreme" Edge

* Wireless gateway is the hub for connected devices (IoT)
* Access to the wireless medium (Wi-Fi, Bluetooth, etc.) &rarr; unique opportunities
* Always-on, low latency, privacy-preserving programmable substrate

![Default-aligned image](figures/extreme-edge.svg)

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
