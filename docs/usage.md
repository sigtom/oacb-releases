# Using OACB

For customer engagement planning and technical handoff, start with the
[field guide for architects and consultants](architects-and-consultants.md).

The first packaged release is still being prepared. There is no published version or image digest to substitute into these instructions yet. An existing OACB operator can already use the YAML workflow below; deployment requires the verified release assets described later in this guide.

## Generate YAML without BMC access

Collect the cluster name and base domain, pull secret, SSH **public** key, networks, API and Ingress VIPs, gateway, DNS, and NTP settings. For each of the three hosts, collect its hostname, static IPv4 address, Linux interface names, MAC addresses, and a stable install-disk hint. Confirm the bond mode, switch configuration, VLANs, and MTU with the customer.

1. Open the operator-provided HTTPS OACB site.
2. Enter the cluster and host information. Leave BMC fields empty.
3. Configure proxy or disconnected mirror settings if required. Optional preflight can be skipped.
4. Select **Validate generated YAML** and resolve the reported errors.
5. Review both `install-config.yaml` and `agent-config.yaml`, especially networks, VIPs, routes, interface membership, rendezvous IP, and each host’s `rootDeviceHints`.
6. Download the validated YAML bundle into a protected working directory.

OACB checks generated host networking with `nmstatectl` and invokes the pinned OpenShift 4.21.27 installer’s `agent create cluster-manifests` command. Validation requires neither BMC credentials nor successful live-network preflight. Redacted previews are review aids; use the deployable files from the private download for installation.

The installer wipes the disk matched by a host’s root-device hint. Prefer a unique WWN or serial number and verify it against the intended hardware before booting. Declared hardware requirements and successful YAML validation do not substitute for that check.

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

## Deploy a packaged release

A [versioned release](https://github.com/sigtom/oacb-releases/releases) supplies a deployment archive, its extraction helper, and a release manifest with checksums and the matching app/proxy image digests. Follow that version’s asset names and checksums. Do not substitute a repository source ZIP, an arbitrary image tag, or files from another release.

For disconnected deployment, download the complete release asset set. Verify `SHA256SUMS` against its hash in the trusted release notes, verify the listed files, then follow the attached `README.md` for offline import and startup. The package contains OACB, not a customer OpenShift payload mirror or pull secret.

Verify the helper and archive checksums against the independently trusted release record, then use the helper’s `extract` command to create the deployment directory. The packaged receipt allows the included tools to verify protected deployment files without a Git checkout. Work from the extracted `deploy/portable` directory.

Use Docker with Docker Compose for online pulls, or rootful Podman with the Docker Compose provider below for online or offline deployment. The target needs Linux AMD64, OpenSSL, curl, standard tar/SHA-256 utilities, and enough memory and disk space for the images and optional ISO workspaces. Select one engine in the shell used throughout this guide:

```sh
export OACB_CONTAINER_ENGINE=docker
docker compose version
```

For Podman, use the distribution's rootful engine and the official [Docker Compose v5.5.1 Linux AMD64 binary](https://github.com/docker/compose/releases/download/v5.5.1/docker-compose-linux-x86_64). The Python `podman-compose` implementation through 1.6.0 lacks commands required by the packaged helpers. Download and transfer the provider before isolation; also install or bring the distribution's Podman packages and runtime/network dependencies, OpenSSL, curl and standard utilities. OACB's image archives do not include these host tools; Docker Engine and Python packages are not needed for this provider.

In a root shell on the target, verify the transferred binary, install it, and select the local [Podman Unix socket](https://docs.podman.io/en/v4.9.0/markdown/podman-system-service.1.html):

```sh
sudo -i
printf '%s  %s\n' \
  'db1889184726840f75c4f9c001048430d4f25b3be3cb084d3ddd762bc0aed576' \
  '/protected/transfer/docker-compose-linux-x86_64' | sha256sum --check - || exit 1
install -D -m 0755 /protected/transfer/docker-compose-linux-x86_64 \
  /usr/local/lib/oacb/docker-compose
systemctl start podman.socket
unset DOCKER_CONTEXT DOCKER_TLS DOCKER_TLS_VERIFY DOCKER_CERT_PATH DOCKER_API_VERSION
unset CONTAINER_HOST CONTAINER_CONNECTION
export OACB_CONTAINER_ENGINE=podman
export PODMAN_COMPOSE_PROVIDER=/usr/local/lib/oacb/docker-compose
export DOCKER_HOST=unix:///run/podman/podman.sock
podman compose version
cd /absolute/path/to/deploy/portable
```

Keep this root shell and environment for loading images, TLS helpers, startup, checks and renewal. Mixing a rootless image load with the rootful socket selects different image stores. Keep the Unix socket restricted to its administrator; do not expose it over TCP or mount it in OACB containers. The packaged guide covers reboot/renewal setup. Check each release's acceptance notes for the host/provider combinations actually verified; the provider pin alone is not native acceptance.

Create candidate settings and private runtime directories:

```sh
mkdir -p runtime/tls runtime/acme
chmod 0700 runtime runtime/tls runtime/acme
install -m 0600 .env.example .env.candidate
export OACB_RELEASE_ENV_FILE="$PWD/.env.candidate"
id -u
id -g
```

Edit `.env.candidate` with:

- The release manifest’s `OACB_APP_IMAGE`, `OACB_NGINX_IMAGE`, `OACB_EXPECTED_REVISION`, and `OACB_DEPLOYMENT_REVISION`. The revisions may differ; copy them exactly.
- Your DNS name in `OACB_PUBLIC_HOST`, desired HTTPS port, and the host interface to publish.
- Your chosen operator access policy and the numeric user/group IDs printed above for the runtime helpers.
- TLS locations and, if necessary, a non-overlapping internal Compose subnet with matching app and proxy addresses.

Public references have the form `ghcr.io/sigtom/oacb-release@sha256:…` and `ghcr.io/sigtom/oacb-release-nginx@sha256:…`. Obtain the complete values from one verified manifest; the ellipses are not usable values. The validator rejects mixed package pairs and mutable tags.

### Choose operator access

The example settings publish only to loopback. Choose the actual IPv4 address assigned to the deployment host. To restrict operators, select `OACB_ADMIN_ACCESS=cidr` and provide their canonical IPv4 networks in `OACB_ADMIN_CIDRS`.

If every peer able to reach the chosen interface is trusted to operate the tool, explicitly select `OACB_ADMIN_ACCESS=trusted-lan`. This mode does not require a CIDR inventory and does not limit customers to private address ranges. It also does not authenticate individual users or infer that a network is trustworthy. Keep the service on an appropriately controlled network.

### Provide HTTPS

The standard path uses a customer-provided certificate chain and matching private key. Set the exact hostname first, then install the files and validate them after pulling the images:

```sh
install -m 0644 /protected/fullchain.pem runtime/tls/fullchain.pem
install -m 0600 /protected/privkey.pem runtime/tls/privkey.pem
```

The included TLS helper also supports generating a CSR for the customer’s PKI team, an explicit short-lived lab self-signed certificate, and optional ACME DNS-01 provisioning. Send only the CSR for signing; never provide a customer CA private key. A self-signed certificate must be trusted by each client, including any BMC using virtual media. The packaged deployment guide contains the commands for each mode.

### Pull and start online

With complete manifest values in `.env.candidate`, pull the public pair anonymously:

```sh
../bin/oacb-image-release validate
../bin/oacb-image-release pull
./bin/validate-release

"$OACB_CONTAINER_ENGINE" compose --project-directory "$PWD" --env-file "$OACB_RELEASE_ENV_FILE" \
  --file "$PWD/compose.yaml" --profile tls-tools run --rm --no-deps tls-tool validate
../bin/oacb-image-release inspect
"$OACB_CONTAINER_ENGINE" compose --project-directory "$PWD" --env-file "$OACB_RELEASE_ENV_FILE" \
  --file "$PWD/compose.yaml" up --detach --no-deps app proxy
"$OACB_CONTAINER_ENGINE" compose --project-directory "$PWD" --env-file "$OACB_RELEASE_ENV_FILE" \
  --file "$PWD/compose.yaml" ps
```

No GitHub username or token is required for public packages. The pull helper checks both digests, architecture, and original image revision. Compose uses already-loaded images and does not pull during startup.

Verify HTTPS from an intended operator workstation, open the UI, and validate a non-sensitive test configuration. Confirm the chosen access policy behaves as intended; a healthy status endpoint alone does not prove operator access. After these checks, retain the tested settings:

```sh
install -m 0600 "$OACB_RELEASE_ENV_FILE" .env
```

For an update, preserve the previous settings and complete image pair before starting the candidate. Follow the packaged rollback procedure if acceptance fails.

### Transfer OACB to an offline network

On a connected staging host with Podman and the same verified deployment files and manifest values, use the rootful engine consistently:

```sh
export OACB_CONTAINER_ENGINE=podman
export OACB_RELEASE_ENV_FILE="$PWD/.env.candidate"
../bin/oacb-image-release pull
install -d -m 0700 offline
OACB_CONTAINER_ENGINE=podman \
OACB_RELEASE_ENV_FILE="$PWD/.env.candidate" \
  ../bin/oacb-image-release export-offline "$PWD/offline"
```

Export uses a pinned Skopeo helper image and therefore also needs staging-host access to that helper image. Public OACB packages need no registry credentials. Transfer the deployment bundle, both OCI archives, and `SHA256SUMS` through the customer’s approved path. Independently record the two archive hashes; a checksum file traveling with the archives is not proof of approval.

On the isolated target, complete the rootful Podman/provider setup above, configure the same release settings, and load the transferred archives in that same root shell with the independently recorded hashes:

```sh
OACB_CONTAINER_ENGINE=podman \
OACB_RELEASE_ENV_FILE="$PWD/.env.candidate" \
  ../bin/oacb-image-release load \
  offline/app.oci.tar '<recorded-app-archive-sha256>' \
  offline/nginx.oci.tar '<recorded-proxy-archive-sha256>'
./bin/validate-release
```

The helper validates both archives before importing into the target image store. With `OACB_CONTAINER_ENGINE=podman` and the provider/socket exports still set, run the TLS validation, image inspection and Compose startup commands from the online section, skipping its pull command. Follow the packaged guide for complete readback and rollback. Docker-only offline import is not a supported MVP path.

This transfers **OACB**, not the OpenShift release payload. A disconnected OpenShift installation still needs its own mirrored content, appropriate pull credentials, trust configuration, and customer network prerequisites. Configure those inputs in OACB and review them with the installation team.

## End the session

Keep generated YAML, ISOs, installer state, pull secrets, kubeconfigs, and admin passwords in approved protected storage. Do not attach them to public reports. Clear the browser session and stop the deployment when the session ends if it is no longer needed. Application artifacts are temporary; retain the exact installer workspace needed to monitor the cluster before stopping it.
