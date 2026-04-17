# Outreachy May 2026: OpenStack Ironic Application
**Applicant:** Kariuki

**Role:** Backend Engineer & Systems Architect

**Location:** Nairobi, Kenya

Welcome to my application repository for the OpenStack Ironic project (Outreachy May 2026). As a backend engineer specializing in Python and custom infrastructure, I approach system design with a logician's mindset. I am passionate about bare-metal provisioning, high-performance systems, and cloud architecture.

This repository serves as a centralized portfolio for my Outreachy application tasks, demonstrating my technical writing, systems debugging, and version control capabilities.

## Quick Links
* [Task 1: Technical Article](https://medium.com/@kariuki_kinyanjui/from-freezing-data-centers-to-cloud-apis-demystifying-openstack-ironic-bb006c918707)
* [Task 2: Bifrost Installation & Debugging Report](https://github.com/kariukikinyanjui/ironic-outreachy-application/blob/main/reports/bifrost_installation_report.md)
* [Task 3: Gerrit Workflow Patch](https://review.opendev.org/c/opendev/sandbox/+/984206)
* [AI Usage Statement](https://docs.google.com/document/d/1emcFUw1RKYd6eowC_y6MAr51IpzEVrS_IP0Z8bfqiPs/edit?usp=sharing)

## Task 1: Demystifying OpenStack Ironic
To demonstrate my technical communication skills, I authored an article breaking down the core concepts of bare-metal provisioning and the architecture of OpenStack Ironic.

* **Read the Article:** [From Freezing Data Centers to Cloud APIs: Demystifying OpenStack Ironic](https://medium.com/@kariuki_kinyanjui/from-freezing-data-centers-to-cloud-apis-demystifying-openstack-ironic-bb006c918707)
* **Focus Area:** The Ironic Control Plane, Conductor handoffs, and the role of the Ironic Python Agent (IPA).

## Task 2: Bifrost Installation Journey
OpenStack Bifrost is a powerful tool, but installing it on custom local hardware with nested virtualization presents unique challenges. I documented my complete installation journey, focusing heavily on the troubleshooting and debugging processes.

* **Read the Full Report:** [Trial by Fire: My Experience Installing and Debugging OpenStack Bifrost](https://github.com/kariukikinyanjui/ironic-outreachy-application/blob/main/reports/bifrost_installation_report.md)

### Highlights from the Report:
Rather than a perfect automated run, my deployment required hands-on architectural intervention. Key debugging scenarios included:
1. **Zombie Processes & Sockets:** Resolving an Nginx `8080` port conflict by investigating socket bindings (`ss -apn`) and spinning up a temporary Python HTTP server.
2. **Global Routing Resets:** Bypassing global firewall/routing drops to `opendev.org` from my ISP by manually cloning dependencies from GitHub mirrors.
3. **The Checksum Infinite Loop:** Diagnosing a deployment timeout by tailing web server logs to catch the Ironic Python Agent (IPA) stuck in an infinite download loop due to a SHA256 checksum mismatch.
4. **Successful Deployment:** Ultimately achieving an `active` node state after correcting the `baremetal-nodes.json` inventory and forcing a node rebuild.
(*Note: The full report includes terminal screenshots and log evidence of the debugging process.)

## Task 3. OpenDev Gerrit Sandbox Patch
To prove my readiness to contribute to the OpenStack ecosystem, I successfully navigated the community's Git workflow. I configured `git-review`, authenticated via SSH, and formatted my commit message to meet OpenStack's strict contribution standards.
* **View my Sandbox Patch:** [patch](https://review.opendev.org/c/opendev/sandbox/+/984206)
* **Commit Details:** Submitted a dummy patch to verify the Gerrit workflow and the automatic generation of the `Change-Id`.

## Live Environment Showcase
To demonstrate my working Bifrost nested virtualization environment, I recorded a brief terminal session navigating the OpenStack CLI, verifying the Ironic Conductor, and inspecting the active bare-metal node.

[![asciicast](https://asciinema.org/a/2gFxAHH5uMIzXWGd.svg)](https://asciinema.org/a/2gFxAHH5uMIzXWGd)

Thank you to the Ironic mentors for taking the time to review my application. Navigating the Bifrost matrix, analyzing database states, and utilizing the Gerrit workflow has been an incredible learning experience, and I am highly motivated to bring this persistence to the OpenStack community.
