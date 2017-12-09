---
title: Quick Start Guide for Using Kubernetes in Rancher
layout: rancher-default-v1.6
version: v1.6
lang: en
---

## Quick Start Guide for Using Kubernetes in Rancher
---

In this guide, you'll learn how to quickly get started using Kubernetes in Rancher v1.6, including:

* Preparing a Linux Host
* Launching Rancher Server (and Accessing the Rancher UI)
* Creating a Kubernetes Environment in Rancher
* Adding Hosts
* Adding Containers
* Launching Catalog Applications

### Preparing a Linux Host

To begin, you'll need to install a [supported version of Docker]({{site.baseurl}}/rancher/{{page.version}}/{{page.lang}}/setup/#supported-docker-versions) on a single Linux host. You can use your laptop, a virtual machine, or a physical server.

>**Note:** Currently, Rancher does not support Docker for Windows and Docker for Mac.

#### To Prepare a Linux Host:

1. Prepare a Linux host with 64-bit Ubuntu 16.04, at least **1GB** of memory, and a kernel of 3.10+.
2. Install a [supported version of Docker]({{site.baseurl}}/rancher/{{page.version}}/{{page.lang}}/hosts/#supported-docker-versions) on the host. To install Docker on the server, follow the instructions from [Docker](https://docs.docker.com/engine/installation/linux/ubuntulinux/).

### Launching Rancher Server

It only takes one command and a few minutes to install and launch Rancher Server. Once installed, you can open a web browser to access the Rancher UI.

> **Note:** We recommend configuring [access control](http://rancher.com/docs/rancher/v1.6/en/configuration/access-control/). Otherwise, your UI and API is available to anyone who can access your IP address.

#### To Launch Rancher Server:

1. Run this Docker command on your host:
   ```bash
   $ sudo docker run -d --restart=unless-stopped -p 8080:8080 rancher/server:stable
   # Tail the logs to show Rancher
   $ sudo docker logs -f <CONTAINER_ID>
   ```
   After launching the container, we'll tail the logs of the container to see when the server is up and running. This process might take several minutes to complete.

2. To access the Rancher UI, go to `http://<SERVER_IP>:8080`, replacing `<SERVER_IP>` with the IP address of your host. The UI displays a Welcome page.

   > **Note:** If you are running your browser on the same host running Rancher Server, you will need to use the host’s real IP address, such as `http://192.168.1.100:8080,` and not `http://localhost:8080` or `http://127.0.0.1:8080`.

3. Click **Authenticate with GitHub**. The Rancher UI displays.

### Creating a Kubernetes Environment in Rancher

Rancher supports grouping resources into multiple [environments]({{site.baseurl}}/rancher/{{page.version}}/{{page.lang}}/concepts/environments/). Each environment starts with an [environment template]({{site.baseurl}}/rancher/{{page.version}}/{{page.lang}}/concepts/environment-template) to define its set of [infrastructure services]({{site.baseurl}}/rancher/{{page.version}}/{{page.lang}}/concepts/infra-services/), and one or more users or groups own it.

In addition to Docker Swarm, Kubernetes, and Mesos, Rancher supports its own container orchestration and scheduling framework called **Cattle**. Rancher uses Cattle extensively to orchestrate infrastructure services, as well as to set up, manage, and upgrade Swarm, Kubernetes, and Mesos clusters.

Initially, Rancher creates a **Default** Cattle environment for you. To deploy Kubernetes in Rancher, you’ll need to configure an environment template and then add the environment to Rancher.

#### To Create a Kubernetes Environment in Rancher:

1. From the **Environment** UI menu, select **Manage Environments**.
2. In **Kubernetes** row of the **Environment Templates** section, click **Edit**.
3. In the **Orchestration** section of the template, click **Edit Config**.
4. Select the version of the template you want to configure, and then enter a **Name** (required) and a **Description** (optional).
5. Adjust the **Configuration Options**, such as RBAC, plane isolation, cloud providers, backups, and add-ons.
6. Click **Configure**, and then click **Save**.
7. From the **Environment** UI menu, select **Manage Environments**.
8. Click **Add Environment**, enter a **Name** (required) and a **Description** (optional).
9. Select your template, and then click **Create**.
10. From the **Environment** UI menu, select your Kubernetes environment. A `Setting up Kubernetes...` message displays and prompts you to add at least one host.

If you configured access control, you can add members to this environment and select their membership roles on the **Manage Environments** page. You can access this new environment from the **Environment** UI menu, or by clicking **Kubernetes > Switch to this Environment**.

### Adding Hosts

The process of adding hosts is the same steps for all container orchestration types, including Cattle and Kubernetes. For Kubernetes, adding your first host deploys infrastructure services, including Kubernetes services like master, kubelet, etcd, and proxy. You can see the progress of the deployment by accessing the **Kubernetes > Infrastructure Stacks** UI menu.

You can either add a host from a cloud provider that Rancher v1.6 supports, or you can add a custom host. If you don't see your cloud provider in the UI, don't worry. Simply use the custom host option.

The `Custom` option is selected by default. It provides the Docker command to launch the Rancher agent container. Rancher uses Docker Machine to launch hosts for the other cloud providers.

If you're adding a custom host, note these requirements:
* Typically, Rancher automatically detects the IP address to register the host.
  * If the host is behind a NAT or the same machine that is running the `rancher/server` container, you might need to explicitly specify its IP address.
* The host agent initiates a connection to the server, so make sure firewalls or security groups allow it to reach the URL in the command.
* All hosts in the environment must to allow traffic between each other for cross-host networking.
  * IPSec: `500/udp` and `4500/udp`
  * VXLAN: `4789/udp`

#### To Add a Host from a Cloud Provider:

1. From the **Infrastructure** menu, select **Hosts**. The Hosts page displays.
2. Click **Add Host**.
3. Enter your **Host Registration URL**, and click **Save**. This URL is where Rancher Server is running, and it must be reachable from any hosts you add. Ignore any warnings for now. The Add Host page displays.
4. Select your cloud provider:
   * Amazon EC2
   * Microsoft Azure
   * DigitalOcean
   * Packet
   * Other

5. Follow the instructions in the Rancher UI to add your host. This process might take a few minutes to complete. Once your host is ready, you can view its status on the Hosts page.

#### To Add a Custom Host:

1. From the **Infrastructure** menu, select **Hosts**. The Hosts page displays.
2. Click **Add Host**.
3. Enter your **Host Registration URL**, and click **Save**. This URL is where Rancher Server is running, and it must be reachable from any hosts you add. Ignore any warnings for now. The Add Host page displays.
4. Select **Custom**. A Docker command displays. For example:
   ```bash
   sudo docker run --rm --privileged -v /var/run/docker.sock:/var/run/docker.sock -v /var/lib/rancher:/var/lib/rancher rancher/agent:v2.0-alpha2
   http://<SERVER_IP>:8080/v3/scripts/D5433C26EC51325F9D98:1483142400000:KvILQKwz1N2MpOkOiIvGYKKGdE
   ```

   >**Note:** The IP address in the command must match your `<SERVER_IP>` and must be reachable from inside your host.

5. Copy, paste, and run the Docker command on your host to register it with Rancher. This process might take a few minutes to complete.
6. Click **Close**. On the Hosts page, you can view the status of your host.

After you add at least one host to your environment, it might take several minutes for all Rancher system services to launch. To verify the status of your environment, from the **Environment** menu, select **Manage Environments**. If a service is healthy, its state displays in green.

### Adding Containers

Once you've verified that all system services are up and running, you're ready to create your first container. To do so, create a pod and then add services to it. A `pod` is one or more containers in Kubernetes.

The process for adding a container differs depending on your container orchestration type. The Rancher UI provides three methods for using Kubernetes. Please refer to Kubernetes' [Overview of kubectl](https://kubernetes.io/docs/reference/kubectl/overview/) for more details.

#### To Add a Container:

1. From the UI menu, click **Kubernetes**, and then select one of the following methods:
   * **Dashboard** &mdash; Access the native Kubernetes dashboard in a new browser window.
   * **CLI** &mdash; Click **Generate Config** to configure `kubectl` for your local machine. Copy and paste the code that displays into your `.kube/config` file, and then run `kubectl`.
   * **kubectl** &mdash; Conveniently access shell in a managed `kubectl` instance right in your web browser.
2. Click **Close** to return to the Rancher UI.   

### Launching Catalog Applications

To help you deploy complex stacks, Rancher offers a catalog of application templates.

#### To Launch a Catalog Application:

1. On the Rancher UI menu, click **Catalog**, and then select one of the following:
   * **All** &mdash; This option displays both Rancher Certified Library and Rancher Community catalog templates.
   * **Library** &mdash; This catalog provides Rancher Certified Library templates for core and infrastructure services. It’s maintained and [certified by Rancher Labs](https://github.com/rancher/rancher-catalog).
   * **Community** &mdash; This catalog provides templates for any application. Templates are created and maintained by the [Rancher Community](https://github.com/rancher/community-catalog), not certified by Rancher Labs.
2. Search for the template you want to launch, and then click **View Details**.
3. Select a **Template Version**. The latest version displays by default.
4. Enter a **Name** (required) and **Description** (optional) for your template.   
5. Enter the **Configuration Options**, which are specific to the template.

    >**Note:** To review the `docker-compose.yml` and `rancher-compose.yml` files used to generate the stacks, click **Preview** before launching the stack.

6. Select **Start services after creating**.
7. Click **Launch**. On the Stack page, you'll see Rancher is creating a stack based on your new application. This process might take a few minutes.

Once your new stack's services are up and running, its state displays in green.

### Next Steps

Now that you've added hosts, created your first container, and launched a catalog application, you can check out the rest of the features in Rancher v1.6:

* [Setup]({{site.baseurl}}/rancher/{{page.version}}/{{page.lang}}/setup/)
* [Concepts]({{site.baseurl}}/rancher/{{page.version}}/{{page.lang}}/concepts/)
* [Tasks]({{site.baseurl}}/rancher/{{page.version}}/{{page.lang}}/tasks/)
* [FAQ]({{site.baseurl}}/rancher/{{page.version}}/{{page.lang}}/faq/)
