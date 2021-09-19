# Accepting the VerneMQ EULA

To use the VerneMQ pre-built packages and Docker images you have to accept the [VerneMQ EULA](https://vernemq.com/end-user-license-agreement). Make sure to read and understand the EULA before accepting it.

## For OS Packages

Accepting the EULA for OS packages can be done by either changing the `accept_eula` line in the `vernemq.conf` file from `no` to `yes` or accepting the EULA the first time starting VerneMQ. In general, the installation of VerneMQ OS packages is now a 3 step process:

1. If you install the package with tools like `dpkg` \(example: `sudo dpkg -i vernemq-1.10.0.xenial.x86_64.deb`\), VerneMQ will install but will fail to start due to the missing EULA acceptance.
2. Accept the EULA by running `sudo vernemq chkconfig` or by adding the following line to your `vernemq.conf file`: `accept_eula = yes`.
3. Start/restart VerneMQ with: `sudo systemctl restart vernemq.`



## For Docker Images

For Docker images the EULA can be accepted by setting the environment variable`DOCKER_VERNEMQ_ACCEPT_EULA=yes`, for Docker Swarm add `DOCKER_VERNEMQ_ACCEPT_EULA: yes` to the environment.

For the Helm chart the EULA for the Docker images can be accepted by extending the `additionalEnv` section with:

`additionalEnv:   
    - name: DOCKER_VERNEMQ_ACCEPT_EULA   
      value: "yes"`

and similarly for the [VerneMQ Operator](../guides/vernemq-on-kubernetes.md#deploy-vernemq-using-the-kubernetes-operator), to accept the EULA for the Docker images, the `env` can be extended with:

`env:   
    - name: DOCKER_VERNEMQ_ACCEPT_EULA   
      value: "yes"`

