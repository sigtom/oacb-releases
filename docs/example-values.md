# Fictional workshop values

Use these examples to walk a customer team through OACB's configuration decisions.
They are **schematic form inputs, not validated installer YAML or a deployable
cluster**. Addresses, domains, interface identities and disk serials are fictional.
They have not passed native installer or installation acceptance as a set.

The first public packaged release is still being prepared. These examples do not
imply that downloadable assets or public images are available. See the
[README](../README.md) for availability and the
[field guide](architects-and-consultants.md) for engagement workflow.

## Shared three-node design

Start with this design for all three scenarios, then apply the changes below.
Documentation addresses and `.test` names deliberately do not identify a customer.

| OACB field | Example value |
| --- | --- |
| Cluster name | `workshop` |
| Base domain | `example.test` |
| OpenShift release | Keep the release's pinned value; current scope is `4.21.27` |
| Architecture | `amd64` |
| FIPS | Disabled for this example |
| Machine network | `192.0.2.0/24` |
| API VIP / Ingress VIP | `192.0.2.20` / `192.0.2.21` |
| Gateway | `192.0.2.1` |
| DNS server | `192.0.2.53` |
| NTP source | `time.example.test` |
| Cluster network / host prefix | `10.128.0.0/14` / `23` |
| Service network | `172.30.0.0/16` |
| Bond mode | Active-backup |
| Machine VLAN ID / MTU | `0` (untagged) / `1500` |
| Rendezvous host | `control-0` |
| Second bond / additional day-2 networking | Disabled |

Use interface names `eno1` and `eno2` on each fictional host. Select **Serial
number** as the root-device hint type and enter the corresponding value below.

| Hostname | Node IP | Primary MAC | Secondary MAC | Root-device value |
| --- | --- | --- | --- | --- |
| `control-0` | `192.0.2.11` | `02:00:00:00:00:01` | `02:00:00:00:00:02` | `WORKSHOP-DISK-001` |
| `control-1` | `192.0.2.12` | `02:00:00:00:01:01` | `02:00:00:00:01:02` | `WORKSHOP-DISK-002` |
| `control-2` | `192.0.2.13` | `02:00:00:00:02:01` | `02:00:00:00:02:02` | `WORKSHOP-DISK-003` |

Leave BMC connection fields empty. If discussing hardware fields, describe a
fictional host with 16 CPU cores, 64 GiB RAM, a 480 GiB install disk and UEFI.
These are example inventory declarations, not sizing guidance or measured disk
performance. Do not invent IOPS, fsync, RAID readiness or firmware evidence to
make validation green. In an engagement, confirm physical interfaces, switch
configuration and the exact disk that installation is authorized to erase.

**Credentials are intentionally omitted.** Use an authorized pull secret and the
operator's actual SSH public key only in the approved execution environment.
Do not paste them into a presentation or completed public example. There is no
fake pull secret, private key or pretend working CA in this document. Without
required valid inputs, expect validation to remain incomplete.

## 1. Connected: customer owns the prerequisites

Set **Installation mode** to **Connected** and **Agent media** to **Full**. Leave
proxy, mirror and additional trust settings empty unless the design requires them.

Walk through naming, addresses, interfaces and install-disk selection. Explain
that manual entry is sufficient for YAML preparation; controller access and live
preflight are optional. Assign DNS, NTP, routing, firewall and registry egress
checks to their owners. For this design, discuss records such as
`api.workshop.example.test`, `api-int.workshop.example.test` and
`*.apps.workshop.example.test`; the example does not create DNS records.

**Handoff:** agreed inputs, resolved native validation results, both reviewed YAML
files and the matching installer version. These fictional values themselves are
not that handoff.

## 2. Proxy and trust: make the network path explicit

Keep **Connected** mode and **Full** media. Add this fictional proxy design:

| OACB field | Example value or required input |
| --- | --- |
| HTTP proxy | `http://proxy.example.test:3128` |
| HTTPS proxy | `http://proxy.example.test:3128` |
| No proxy | `localhost,127.0.0.1,.example.test,.svc,.cluster.local,192.0.2.0/24,10.128.0.0/14,172.30.0.0/16` |
| Additional trusted CA bundle | Customer-provided PEM CA certificates, if required; no sample certificate supplied |

The HTTPS proxy field can use an HTTP URL for an HTTP CONNECT proxy. The scheme
identifies the connection to the proxy, not the final destination protocol.
Adapt exclusions to the approved site design rather than copying this list
unchanged. This example deliberately keeps cluster-local traffic off the proxy.

If the proxy inspects TLS, obtain the correct CA chain through the customer's PKI
process. If authentication is required, arrange secure credential delivery;
do not embed credentials in this example URL. Validate the configuration, then
verify proxy reachability and trust from the actual installer and node networks.
A browser form accepting these values does not prove either path works.

## 3. Disconnected: separate the tool from installation content

Change **Installation mode** to **Disconnected**. Keep **Full** media for the
simplest discussion, or select **Minimal** to explain external boot artifacts.

| OACB field | Schematic value |
| --- | --- |
| Mirror registry | `registry.example.test:8443` |
| Release source repository | `quay.io/openshift-release-dev/ocp-release` |
| Corresponding mirror repository | `registry.example.test:8443/openshift/release-images` |
| Component source repository | `quay.io/openshift-release-dev/ocp-v4.0-art-dev` |
| Corresponding mirror repository | `registry.example.test:8443/openshift/release` |
| Additional trusted CA bundle | Actual registry CA chain from the customer's PKI process, if required |
| Boot artifacts base URL, for Minimal media | `https://boot.example.test/rhcos/` |

Mirror mappings are examples of field syntax only. Use the **actual mappings
produced by the customer's mirroring operation**, with the content for the
matching OpenShift release and architecture. Repository fields take
`host[:port]/path`, without `https://`; the boot-artifact field takes a URL.
Do not assume these illustrative paths contain payloads or the required RHCOS
boot artifacts.

Account separately for the OACB runtime/image bundle, mirrored OpenShift content,
and installation network services. OACB's offline delivery does not create a
registry mirror or provide pull credentials. Minimal media also needs a reachable,
trusted boot-artifact location with matching content. Keep proxy fields empty
unless the actual disconnected design includes a proxy.

**Handoff:** confirmed mirror mappings, registry trust and credential arrangements,
content/architecture evidence, boot-artifact prerequisites where applicable, and
the reviewed YAML. Record unresolved dependencies in the
[engagement worksheet](engagement-worksheet.md).

## Demonstration finish

Use the inputs to explain **configure → validate → review → download**. Native
validation should be demonstrated only in an approved runtime with complete,
valid inputs; otherwise show the intended steps and explain the missing inputs.
Do not present these tables as validated output or attempt BMC boot operations
against fictional targets. Generated customer YAML and installer state stay in
approved protected storage, outside this repository.
