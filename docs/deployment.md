# Deploy OACB on a shared or disconnected host

For a local demonstration or YAML generation, use the [quickstart](usage.md).
This guide is for administrators setting up the app and HTTPS proxy for other
operators, or transferring OACB into a disconnected network.

| You need to… | Start here |
| --- | --- |
| Set up a shared HTTPS site | [Configure the deployment](#deploy-a-packaged-release) |
| Install without registry access | Download all twelve files from [v0.1.0](https://github.com/sigtom/oacb-releases/releases/tag/v0.1.0) and follow its attached **README.md**. The container archives are already prepared. |
| Create a new offline export yourself | [Export and transfer](#transfer-oacb-to-an-offline-network) — optional when using the published archives |
| Generate installer YAML | [Use OACB](usage.md#generate-yaml-without-bmc-access) |

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

