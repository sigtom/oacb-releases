# Create installation media and boot the cluster

Start with the [YAML quickstart](usage.md) to generate and download
`install-config.yaml` and `agent-config.yaml`. Follow this guide when you are
ready to create an ISO or use supported BMC boot operations.

The current release bundles OpenShift Installer **4.21.27**. Customer DNS,
switching, disks and OpenShift release-content access must be ready before boot.
For disconnected installs, prepare the OpenShift mirror, pull credentials and
trust separately from OACB's offline application bundle. FIPS configurations need
validation with the same installer on a FIPS-enabled RHEL system.

## Create an ISO on a workstation

Use the same `openshift-install` **4.21.27** release and have `nmstatectl` on the workstation’s `PATH`. Obtain tools through your organization’s approved channel. Check their versions before copying customer data:

```sh
openshift-install version
nmstatectl --version
```

Copy the two reviewed files into a fresh protected directory; replace these example paths with your approved local locations:

```sh
install -d -m 0700 /protected/cluster-config
install -m 0600 /protected/transfer/install-config.yaml /protected/cluster-config/
install -m 0600 /protected/transfer/agent-config.yaml /protected/cluster-config/
openshift-install --dir /protected/cluster-config agent create image
```

For AMD64, the output is `agent.x86_64.iso`. Keep the **whole installer directory**, including its state and authentication files. After mounting the ISO through your approved method and booting the three intended hosts, monitor from that same directory:

```sh
openshift-install --dir /protected/cluster-config agent wait-for bootstrap-complete
openshift-install --dir /protected/cluster-config agent wait-for install-complete
```

OACB can also create an ISO in the application. Download any offered installer-workspace handoff promptly: it is a one-use, short-lived credential-bearing archive. A restart or expiry removes temporary application artifacts.

## Optional discovery and BMC boot

Read-only Redfish discovery lets you review hardware before importing selected values. It does not silently change host configuration. The operator verifies controller trust and supplies credentials for the request.

BMC changes are disabled by default. Supported actions require fresh discovery, a preview, an exact typed confirmation, and a final readback. Capabilities depend on the controller and firmware; a successful discovery does not establish support for all actions.

For application-hosted AMD64 media, the BMC must reach the configured HTTPS endpoint and accept its certificate. Media access is restricted to the explicitly selected BMC source addresses and expires. Verify the observed source from each BMC network path using the bundled source-address check before using this feature. Do not add BMC networks to operator access merely to allow media reads.

YAML generation and workstation ISO creation remain available when this BMC path cannot be used.

Shared access, application-hosted BMC media and the secure installer-workspace
handoff use the [HTTPS deployment](deployment.md). The local quickstart is for
YAML preparation and demonstrations.

## End the session

Keep generated YAML, ISOs, installer state, pull secrets, kubeconfigs, and admin passwords in approved protected storage. Do not attach them to public reports. Clear the browser session and stop the deployment when the session ends if it is no longer needed. Application artifacts are temporary; retain the exact installer workspace needed to monitor the cluster before stopping it.
