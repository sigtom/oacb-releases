# OACB quickstart

Start the app, enter your cluster details, then validate and download the YAML.
You do not need BMC access, GitHub credentials or an application-source checkout.

**Already have an OACB site?** Open it and skip to
[Generate YAML](#generate-yaml-without-bmc-access).

## Start OACB locally

Use a **Linux x86-64 machine with Podman installed, 4 CPUs and 8 GiB RAM**.
On an Apple Silicon Mac or Windows workstation, use a suitable Linux host for
this release.

Copy this command to start **v0.1.0**. Podman downloads the image automatically
the first time:

```sh
podman run --rm --name oacb \
  --read-only --cap-drop=all --security-opt=no-new-privileges \
  --pids-limit=256 --memory=6g --cpus=4 \
  --tmpfs /tmp:rw,noexec,nosuid,nodev,size=512m \
  --tmpfs /work:rw,noexec,nosuid,nodev,size=4g \
  --publish 127.0.0.1:8080:8080 \
  ghcr.io/sigtom/oacb-release@sha256:b147f7e053cc8415b43463b6520a115103f6a5cf1cf4b9359d97e9db8a72759a
```

Leave that terminal running. Open **[http://127.0.0.1:8080](http://127.0.0.1:8080)**
in a browser on the same machine.

<details>
<summary>Using a remote Linux host?</summary>

Keep OACB running there. In another terminal on your workstation, open an SSH
tunnel, replacing `user@linux-host` with your SSH destination:

```sh
ssh -N -L 8080:127.0.0.1:8080 user@linux-host
```

Then open `http://127.0.0.1:8080` in your workstation's browser.

</details>

This local session is for YAML preparation and demonstrations. It listens only
on loopback. For a shared site or BMC media/boot workflow, use the
[HTTPS deployment guide](deployment.md).

## Generate YAML without BMC access

For a demo, start with the [fictional example values](example-values.md).
For customer work, have the cluster name/domain, pull secret, SSH public key,
networks and three host inventories ready. The [worksheet](engagement-worksheet.md)
helps collect those values.

1. **Enter the cluster settings:** networks, VIPs, DNS and NTP. Add proxy or
   mirror settings only when the environment needs them.
2. **Enter the three hosts:** names, IPs, interfaces, MAC addresses and install
   disks. Leave BMC fields empty when entering inventory manually.
3. **Select Validate generated YAML.** Fix any reported errors; optional
   network preflight is not required.
4. **Review both files:** `install-config.yaml` and `agent-config.yaml`.
   Check the host networking and disk selection against the intended machines.
5. **Download the validated bundle** into a protected working directory.
   Use its deployable files for installation, rather than the redacted previews.

Before booting, verify each install-disk hint: the installer wipes the matching
disk. YAML validation does not prove the customer's physical network or a
complete cluster installation.

## Next steps

- **Create an Agent ISO or boot through a BMC:** [Installation guide](installing-clusters.md).
- **Share OACB with other operators:** [HTTPS deployment](deployment.md).
- **Run OACB offline:** download the complete [release asset set](https://github.com/sigtom/oacb-releases/releases/tag/v0.1.0)
  and follow its attached **README.md**. Both container images are included;
  the customer's OpenShift mirror and pull credentials are separate.

When finished, download everything you need and press **Ctrl+C** in the OACB
terminal. Inputs and generated artifacts are temporary. Keep customer files
and credentials out of public issues and Git repositories.
