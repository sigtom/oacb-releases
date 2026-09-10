# Field guide for Red Hat architects and senior consultants

OACB helps turn an agreed customer infrastructure design into reviewed Agent-based
Installer inputs. Use it during discovery, implementation planning, a build
workshop, or handoff to a customer installation team. The intended first audience
is Red Hat architects and senior consultants working on customer environments.

The aim is to reduce the engagement-specific scripting needed to collect host
information, populate installer YAML, create media and prepare supported BMC boot
operations. It does not replace architecture decisions, customer infrastructure
preparation, or an engagement's change and acceptance process. Existing Ansible
and site automation can remain responsible for those tasks.

The first public packaged release is being prepared. See the [README](../README.md)
for release availability and current coverage before planning a customer build.
This is an independent project, not a statement of Red Hat product support or
endorsement.

## Choose the engagement path

| Customer situation | How to use OACB | Deliverable |
| --- | --- | --- |
| No BMC access; customer provides inventory and network design | Enter the agreed values manually, run native YAML validation and review the result | Deployable `install-config.yaml` and `agent-config.yaml`, validation evidence and an installation handoff |
| OACB cannot run inside the customer environment | Use an approved preparation environment if customer data handling permits, or demonstrate with fictional inputs; the customer runs the matching installer in its approved environment | Reviewed inputs and a clear list of checks the customer must perform on site |
| Disconnected installation | Prepare mirror, trust and proxy-exclusion inputs; deliver OACB's offline bundle separately from the OpenShift mirrored content | Configuration plus separate tool, content and connectivity prerequisites |
| Read-only controller access is available | Review Redfish inventory and explicitly apply selected values to the form | An evidence-backed host plan; discovery does not authorize changes |
| ISO hosting and supported BMC actions are permitted | Generate the ISO, establish HTTPS/media reachability, then preview and confirm individual supported actions | Installation media, retained installer state and action readbacks |

YAML generation is a complete workflow of its own. Do not make a customer provide
BMC credentials or pass optional live preflight just to prepare installer inputs.

## Before the build workshop

Use the [engagement worksheet](engagement-worksheet.md) to record decisions,
evidence, owners and unresolved items. Keep a completed copy in the customer's
approved location, not in this public repository.

Confirm the intended OpenShift release and topology against OACB's current
coverage. The current MVP targets three compact bare-metal nodes with static
IPv4; the pinned installer is 4.21.27. A selectable setting is not proof that every
combination has installation acceptance coverage. Resolve a version or topology
mismatch before promising a delivery date.

Identify who owns DNS, address allocation, switching, storage/RAID readiness,
registry mirroring, PKI, firewall changes and BMC access. Record whether each host
value came from controller discovery, an inventory export or a customer
declaration. Leave unknowns unresolved rather than treating a default as evidence.

Agree where OACB may run, how customer inputs may be transferred, and who will
retain the generated artifacts. OACB has no built-in user accounts: use the
customer's approved HTTPS and network access policy. Its temporary workspaces are
not an engagement record or a backup.

## Run the configuration review

1. Enter the agreed cluster and network design, then add the three host plans.
   Import discovery results only after reviewing the selected controller/system
   and the specific values being applied.
2. Check bond members and mode against switch configuration; confirm VLANs,
   MTU, routes, DNS, NTP, VIPs and rendezvous selection with their owners.
3. Verify each install disk by a stable hardware identifier. Root-device hints
   identify disks that the installer may erase; capacity alone is not a unique
   disk selection or proof of RAID readiness.
4. Run **Validate generated YAML**. Resolve field, NMState and installer
   manifest-validation errors, then review both output files with a peer.
5. Use optional preflight from the relevant network location. Capture whether a
   result is passed, failed or inconclusive and what it actually observed.
6. Download the deployable bundle into protected storage. Record the selected
   OACB release, matching installer version, review date and remaining prerequisites.

The [usage guide](usage.md) contains the operator steps and installer commands.
Do not use redacted review examples as installation inputs.

## Explain what validation proves

| Evidence | What it establishes | What remains to verify |
| --- | --- | --- |
| OACB field and cross-field checks | Inputs meet the implemented configuration rules | Accuracy of customer declarations |
| Native NMState validation | Generated host networking is accepted by the bundled validator | Interface identity, physical links and switch behavior |
| `openshift-install agent create cluster-manifests` | The matching installer accepts the generated configuration at this stage | ISO creation, boot, network/content access and installation completion |
| Optional network preflight | Observations from the probe's actual network location | Paths from other networks, node-to-node behavior and checks reported inconclusive |
| Redfish inventory/readback | Reported state of the selected controller/system at the time of inspection | Undiscovered properties and changes made after inspection |
| Installation monitoring and agreed acceptance tests | Outcomes of the actual installation and customer acceptance process | Workloads and operational requirements outside those tests |

A green validation result is useful pre-install evidence, not a promise that the
cluster will install. End-to-end real-hardware acceptance has not yet been
established for the public release. Keep that limitation explicit in proposals,
demos and handoffs.

## Disconnected delivery: separate the three requirements

- **OACB runtime:** deployment files, the matching app/proxy image pair and their
  verified checksums. The current offline image import path uses Podman.
- **OpenShift installation content:** the customer's mirrored release content,
  credentials and trust configuration. Transferring OACB does not create this mirror.
- **Installation network:** working DNS, time, routing and access from the nodes
  and installer environment to the services they need. An isolated application
  host alone does not demonstrate these paths.

Prepare and test the actual transfer method with the customer. Omit optional
internet-dependent certificate provisioning in a fully isolated workflow; use
the customer's certificate process. YAML-only preparation can proceed without
controller access, but unresolved infrastructure prerequisites still belong in
the installation handoff.

## Handoff to the installation team

Provide the reviewed deployable YAML through the approved secure channel, the
matching installer version and commands, the host-to-disk/network mapping, and
an owned list of remaining prerequisites. Include relevant validation evidence
with customer data protected. State who is authorized to create media, attach it,
boot the hosts and monitor the installation.

If an ISO was created, retain the complete installer directory needed for
`agent wait-for bootstrap-complete` and `agent wait-for install-complete`, including
its state and authentication files. When OACB supplies the one-use workspace
handoff, download it promptly and verify receipt before ending the session.
Do not rely on a temporary media URL or OACB workspace as long-term storage.

Close the activity with actual installation outcomes and the customer's agreed
acceptance evidence. Keep follow-up configuration, workload onboarding and day-2
operations visible as separate deliverables.

## Demonstrate it to a colleague

Start with the [fictional workshop values](example-values.md) for a three-node
connected, proxy/trust or disconnected design. Show manual entry with BMC fields
empty, native validation, review of both YAML files, and the protected download
step. Then explain where optional discovery and ISO/BMC actions fit. If no
approved runtime or release is available, walk through the design and guide
without implying a live installation or a downloadable release exists.
