# OpenShift Agent Configuration Builder

OACB is built for architects and senior consultants preparing OpenShift installations in customer environments. Its first audience is Red Hat field teams. It helps you turn a customer’s cluster plan into reviewed OpenShift Agent-based Installer configuration. Enter the network and host information in a browser, validate the generated YAML with the pinned installer, then download it for the installation team.

**BMC access is optional.** You can generate `install-config.yaml` and `agent-config.yaml` without contacting a server controller. When access is available, OACB can also help discover hardware, create an Agent ISO, and prepare supported virtual-media boot operations.

This repository is the public home for release downloads, deployment instructions, and examples. The first packaged release is being prepared; no downloadable release or public image pair is available yet. The instructions below describe the release workflow and will become usable when its verified assets are published.

## What you can do

- **Prepare configuration:** enter cluster, host, disk, bond, VLAN, routing, DNS, and NTP settings; review and download both installer YAML files.
- **Validate before handoff:** check field relationships, validate each host’s network configuration with NMState, and run the matching OpenShift Installer’s manifest validation.
- **Plan for customer networks:** supply connected, proxy-connected, or disconnected mirror and certificate settings.
- **Use hardware discovery when available:** inspect Redfish inventory and explicitly select which discovered values to apply.
- **Create installation media:** generate a full or minimal Agent ISO, or take the YAML to a workstation with the matching installer.
- **Use guarded BMC actions where supported:** review and confirm individual virtual-media and boot operations. Mutation controls are off by default.

## Current scope

The current MVP targets a three-node compact bare-metal OpenShift **4.21.27** cluster with static IPv4 networking and a disabled provisioning network. The portable application image runs on Linux AMD64.

Successful YAML validation does not prove that the customer’s DNS, switches, disks, firmware, or registry mirrors are correct. A complete real-hardware cluster installation has not yet been established as release acceptance evidence. Other configurations and vendor-specific BMC features should not be assumed to have the same coverage.

## Get started

Start with the [field guide for Red Hat architects and senior consultants](docs/architects-and-consultants.md) for engagement planning, design review and customer handoff. Use the [engagement worksheet](docs/engagement-worksheet.md) to collect decisions and assign unresolved prerequisites.

Use the [fictional workshop values](docs/example-values.md) for connected, proxy/trust, and disconnected design walkthroughs. They are schematic inputs, not validated installation outputs.

The [usage guide](docs/usage.md) covers the operator steps: YAML-only preparation, optional ISO creation, and connected or offline deployment.

Each packaged release is intended to include the deployment files, verification helper, checksums, and a manifest identifying the matching images:

- `ghcr.io/sigtom/oacb-release`
- `ghcr.io/sigtom/oacb-release-nginx`

Use the **two immutable digests from one release manifest**, not `latest` or a mixed pair. Public image pulls will not require a GitHub token. Once the release bundle is published, you will not need an application-source checkout to deploy it.

## License and release source

OACB's original code, scripts, documentation and examples are licensed under
[Apache-2.0](LICENSE). See [NOTICE](NOTICE) for attribution and the distinction
between OACB and bundled third-party software, which retains its own licenses.

Each packaged release will include an editable `oacb-source.tar` with the
application source, dependency locks, build recipes and build instructions.
The release manifest will bind the archive checksum and the exact source
revision used to build the matching images. Verify the checksums from your
trusted release acquisition channel before extracting an archive. Building
from source requires the documented build dependencies; the offline runtime
bundle does not require a rebuild.

GitHub's automatically generated **Source code (zip/tar.gz)** downloads contain
this documentation repository. Choose the attached **oacb-source.tar** for the
application source. The development repository, history and CI remain private;
public release source archives will contain the inputs needed to edit and build
the application. The first source archive will be available with the first
packaged release.

## Customer data and access

OACB is designed for an operator-controlled network. It has no built-in user accounts or role-based access; deploy it behind HTTPS and choose the appropriate network access policy. Inputs and generated installer workspaces are temporary, so download the artifacts you need before ending the session.

Never publish pull secrets, BMC credentials, customer YAML, certificates, generated ISOs, kubeconfigs, or installer state in this repository. Use sanitized descriptions for questions and reports.

OACB is an independent project. It does not replace Red Hat’s installation prerequisites or support policies, and no Red Hat endorsement is implied.

For security concerns, follow the [private reporting instructions](SECURITY.md).
