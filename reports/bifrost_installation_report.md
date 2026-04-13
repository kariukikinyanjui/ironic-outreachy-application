# Trial by Fire: My Experience Installing and Debugging OpenStack Bifrost
**Author:** Kariuki Kinyanjui

**Role:** Backend Engineer & Systems Architect

**Environment:** Ubuntu 24.04 LTS (Nested Virtualization via KVM/QEMU)

## Overview
OpenStack Bifrost is a set of Ansible playbooks that automates the deployment of a standalone OpenStack Ironic bare-metal provisioning environment. While automated tools are powerful, real-world infrastructure often presents unique environmental challenges.

This report documents my hands-on experience installing Bifrost on local hardware using nested virtualization. Rather tha presenting a sanitized, perfect run, this document highlights the actual networking, state-machine, and routing hurdles I encountered, alongside the native Linux debugging tools I used to overcome them.

## Challenge 1: The Port 8080 Socket Conflict

**The Problem**
During the execution of the `test-bifrost.sh` deployment script, the Ansible playbook threw a fatal error while attempting to configure and start the `nginx` service for the `httpboot` directory. Nginx report an `[emerg] bind () to 0.0.0.0:8080 failed (98: Address already in use`) error.

**The Investigation**
Ansible abstracts away system-level conflicts, so I dropped down to the Linux command line to investigate socket bindings. I utilized the `ss` (socket statistics) utility to track down the rogue process hoarding the port:

```
sudo ss -apn | grep 8080
```

**The Evidence**

![Terminal output proving a background Python3 process (PID 617289) was actively squatting on port 8080, preventing Nginx from binding](https://github.com/kariukikinyanjui/ironic-outreachy-application/blob/main/assets/01-nginx-port-conflict.png)

**The Resolution**
I identified the process as a stale Python HTTP server from an earlier manual test. I terminated the process using `sudo fuser -k 8080/tcp`, freeing the socket and allowing the Ansible playbook to successfully bind Nginx on its next run.

## Challenge 2: The Global Routing Drop

**The Problem**
As the Bifrost playbook progressed to the repository cloning phase, the process suddelnly halted. The deployment script threw a fatal Git fetch error, terminating the playbook entirely.

**The Investigation**
Analyzing the Ansible JSON traceback revealed that the `git fetch` module failed with `Failed to connect to opendev.org port 443 after 2ms: Couldn't connect to server`. Further network tracing indicated that my ISP was actively dropping or resetting specific routing paths to OpenDev's infrastructure.

**The Evidence**

![The raw Ansible traceback showing the TCP connecting reset when attempting to clone from opendev.org.](https://github.com/kariukikinyanjui/ironic-outreachy-application/blob/main/assets/02-opendev-connection-reset.png)

**The Resolution**
To bypass the ISP-level routing restriction, I manually intervened by adjusting the Ansible playbook configuration to clone dependencies from alternative GitHub mirrors where necessary, allowing the environment setup to proceed.

## Challenge 3: The Ghost Image and the Infinite Loop

**The Problem**
The most complex issue occurred during the actual bare-metal provisioning phase. The virtual machine (`testvm`) powered on via IPMI, loaded the Ironic Python Agent(IPA) into RAM, but then became permanently stuck in the `deploying`(wait call-back) state until the deployment eventually timed out.

**The Investigation**

OpenStack Ironic provides high-level state views, but to understand what the node was actively doing, I needed to look at the web server serving the deployment images. I tailed the Nginx/HTTP access logs to monitor incoming requests from the IPA ramdisk.

```
tail -f /var/lib/ironic/httpboot/httpboot_loop.log
```
I discovered the agent was trapped in an infinite loop, constantly requesting `deployment_image.qcow2`. It was downloading the image, checking the SHA256 hash, failing the validation, discarding the image, and asking for it again.

**The Evidence**

![The OpenStack Baremetal Node List showing the node hopelessly stalled in the "deploying" state.](https://github.com/kariukikinyanjui/ironic-outreachy-application/blob/main/assets/04-httpboot-infinite-loop.png)

---
![Tailing the web server logs revealed the smoking gun, the Ironic Python Agent repeatedly requesting the image due to a checksum validation failure.](https://github.com/kariukikinyanjui/ironic-outreachy-application/blob/main/assets/03-ironic-node-timeout.png)

**The Resolution**
The root cause was traced to a mismatch between the actual `qcow2` image hash and the `image_checksum` registered in the Ironic MariaDB database. I manually overrode the database entry using the OpenStack CLI, rebuilt the node, and watched the agent successfully validate the image.

## The Final Victory
After clearing the socket conflict, bypassing the firewall routing, and correcting the Ironic database checksums, the final deployment succeeded flawlessly. The node progressed cleanly through the `deploying` state, requested its `configdrive`, and settled into a healthy, operational state.

![The final `openstack baremtal node list` showing testvm1 in the target `active` state with the power on](https://github.com/kariukikinyanjui/ironic-outreachy-application/blob/main/assets/05-successful-deployment.png)

**Conclusion**
Deploying Bifrost is an excellent exercise in full-stack debugging. This journey required navigating Ansible abstractions, managing Linux systemd services, utilizing native networking tools, and interacting directly with the OpenStack CLI and state machine. These challenges ultimately provided a much deeper understanding of the Ironic control plane than a frictionless installation ever could.
