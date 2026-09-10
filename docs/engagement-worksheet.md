# Customer engagement worksheet

Use this alongside the [architect and consultant field guide](architects-and-consultants.md).
Copy it into an approved engagement workspace. This blank worksheet is not an
OACB import file. Do not commit a completed copy, credentials or generated
customer artifacts to this repository.

## Scope and ownership

- Engagement / design reference:
- Architect or technical lead:
- Customer installation and acceptance owner:
- Intended OpenShift release, architecture and topology:
- OACB release and pinned installer compatibility confirmed by:
- Delivery path: YAML only / YAML plus ISO / optional BMC assistance:
- Connected / proxy-connected / disconnected:
- Approved OACB execution location and operator access policy:
- Approved artifact transfer, retention and cleanup process:
- Installation change window and authorization owner:

## Design inputs

Record the approved reference and owner, not secret values, in this worksheet.

| Area | Required decisions or evidence | Owner / reference / open items |
| --- | --- | --- |
| Naming and addressing | Cluster/base domain, hostnames, machine/cluster/service networks, API and Ingress VIPs, address reservations | |
| Host networking | Physical interface names and MACs, bond membership/mode, VLANs, MTU, gateway and routes, switch configuration | |
| Core services | DNS records/resolution paths, NTP sources, routing and firewall prerequisites | |
| Host storage | Stable install-disk identifier for each host, intended disk confirmation, RAID readiness and capacity/performance evidence | |
| Host readiness | CPU/RAM, architecture, boot/firmware settings and any requested hardware capabilities | |
| Registry and proxy | Connected or mirror endpoints, mirror-content readiness, proxy/exclusions, trust and credential delivery owner | |
| Operator access | HTTPS name/certificate, host bind, CIDR policy or explicitly trusted LAN, network reachability | |
| Optional controller access | Authorized targets, controller/firmware, selected system, trust process, permitted actions and media network path | |
| Tooling | OACB image pair and deployment bundle, matching installer, offline transfer/import where needed | |

## Host plan

| Host | Role | Address / interface-plan reference | Install-disk identifier reference | Inventory source and date | Unresolved items |
| --- | --- | --- | --- | --- | --- |
| Host 1 | control plane | | | | |
| Host 2 | control plane | | | | |
| Host 3 | control plane | | | | |

Identify customer-declared, discovered and independently verified values. Do not
replace an unknown interface, disk or controller capability with an assumed value.

## Review and handoff

- Peer review of both generated YAML files completed by / date:
- Native NMState and installer validation result / protected evidence location:
- Optional preflight result, observation location and inconclusive checks:
- Confirmed install-disk mapping and authorization to boot:
- Outstanding prerequisites, each with an owner and expected resolution:
- Secure YAML / ISO / installer-state handoff recipient and receipt confirmation:
- Installation monitoring owner and retained installer directory location:
- Agreed cluster acceptance criteria and evidence location:
- Final result, follow-up work and artifact cleanup confirmation:

Record pull-secret, BMC, proxy and registry credential **owners and delivery
arrangements**, never their values. Installer state and authentication files
belong in protected storage, not in an ordinary meeting note or public issue.
