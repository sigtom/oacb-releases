# OpenShift Agent Configuration Builder

OACB helps architects and consultants turn a customer's cluster plan into
validated OpenShift Agent-based Installer YAML. Enter the network and host
details in a browser, review the generated files, and download them for the
installation team.

**[Start here: run OACB and generate YAML](docs/usage.md)**

The quickstart uses one prebuilt container on a Linux x86-64 host. No source
build, GitHub login or BMC access is required.

## Choose your next step

| I want to… | Guide |
| --- | --- |
| Try OACB or generate YAML | [Quickstart](docs/usage.md) |
| Use example inputs for a demo | [Fictional example values](docs/example-values.md) |
| Create an ISO or use supported BMC boot operations | [Installation guide](docs/installing-clusters.md) |
| Set up a shared HTTPS site | [Deployment guide](docs/deployment.md) |
| Install OACB in a disconnected network | [Download the release](https://github.com/sigtom/oacb-releases/releases/latest) and follow its attached **README.md** |
| Plan a customer engagement | [Architect and consultant field guide](docs/architects-and-consultants.md) and [worksheet](docs/engagement-worksheet.md) |

## What OACB does

- Generates and validates `install-config.yaml` and `agent-config.yaml`.
- Supports connected, proxy-connected and disconnected configuration.
- Accepts inventory entered manually or imported from reviewed Redfish discovery.
- Can create Agent installation media and help with explicitly enabled,
  supported BMC media and boot operations.

The current MVP targets a three-node compact bare-metal cluster with static
IPv4, a disabled provisioning network and OpenShift **4.21.27**. The published
application runs on **Linux AMD64**.
YAML validation checks the configuration; real hardware, customer connectivity
and a complete cluster installation require their own verification.

## Downloads and source

**[Download the latest release](https://github.com/sigtom/oacb-releases/releases/latest)**
for runnable app/proxy images, offline container archives, deployment files,
checksums and editable application source. Everything is publicly downloadable.

The shared HTTPS deployment uses the matching `oacb-release` and
`oacb-release-nginx` images from one release manifest. The local quickstart
uses the application image alone.

To build or change the application, choose the attached **`oacb-source.tar`**.
GitHub's automatic **Source code (zip/tar.gz)** downloads contain this
documentation repository. Development history and CI remain private.

Original OACB code, scripts and documentation use [Apache-2.0](LICENSE);
see [NOTICE](NOTICE). Bundled third-party software retains its own terms.
OACB is an independent project; no Red Hat endorsement or product support is implied.

## Customer data

OACB has no built-in user accounts. The quickstart stays on localhost;
shared deployments need HTTPS and an appropriate network access policy.
Download the artifacts you need before stopping the app: its workspaces are temporary.

Keep customer YAML, pull secrets, BMC credentials, certificates, ISOs,
kubeconfigs and installer state in protected storage. Use fictional, redacted
examples in public issues. Report security concerns through the
[private reporting instructions](SECURITY.md).
