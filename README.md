# palette-appliance-automation

Unattended build and deploy of a self-hosted [Palette](https://docs.spectrocloud.com/enterprise-version/)
management appliance: builds an installer ISO with
[CanvOS](https://github.com/spectrocloud/CanvOS), installs it on a KVM/libvirt
VM, uploads the Palette Enterprise content bundle, and deploys the management
cluster.

```bash
git clone --recurse-submodules <this-repo>
cd palette-appliance-automation
cp /path/to/downloads/* artifacts/        # see artifacts/README.md
ansible-playbook site.yml
# → https://<vip>         tenant console
# → https://<vip>/system  system console
```

Already cloned without submodules: `git submodule update --init`.

## Layout

```
site.yml                 three stages, tagged build / vm / cluster
group_vars/all.yml       every input and knob
artifacts/               your Artifact Studio downloads (gitignored)
CanvOS/                  submodule, pinned; patched in place, never committed
roles/
  canvos_iso/            renders build inputs, builds the ISO
  appliance_vm/          VIP reservation, disks, unattended install, first boot
  palette_cluster/       upload, cluster definition, create cluster, wait
```

Run one stage: `--tags build`, `--tags vm`, `--tags cluster`.

## Inputs

Put your downloads in `artifacts/` (auto-discovered by extension, see
[artifacts/README.md](artifacts/README.md)): the content bundle, its detached
signature, and the content-signing public key.

Then review `group_vars/all.yml` — in particular `pe_version`, which must match
the agent version your content bundle ships, and `cluster_vip`, which must be
free on your libvirt network.

## Requirements

`ansible-core`, `libvirt` + `virt-install`, `qemu-kvm`, OVMF, Docker (for the
CanvOS build), plus `curl`, `zstd`, `openssl`, `xxd`, `sshpass`, `bsdtar`.

Follow Spectro Cloud's published sizing for the appliance: per node 8 vCPU,
16 GB RAM, a 300–500 GB install disk, and a second disk of at least 500 GB for
the storage pool, which is wiped during deployment. Single node is for testing;
production deployments use three control-plane nodes.

## Operating notes

- **Unattended by design.** The playbook seeds the appliance's administrative
  account during installation, so no console interaction is required. Set
  `appliance_admin_user` / `appliance_admin_password`, or let the play generate
  a password and print it in the final summary.
- **Single-node overrides.** The shipped profiles default to three replicas for
  storage placement and the database; `csi_placement_count` and `mongo_replicas`
  set those to 1 for a single-node deployment.
- **Generated credentials.** The internal registry password is generated per
  run unless you set `registry_password`, and is shown in the summary. Treat
  the summary output as sensitive.
- **Long-running.** A full run is roughly an hour: ISO build, install, a ~10 GB
  upload, then 20–30 minutes of cluster deployment. Timeouts are tunable in
  `group_vars/all.yml`.
- **Virtualization details.** The VM uses UEFI without Secure Boot, host CPU
  passthrough, virtio-scsi disks, and VGA video; the console does not render
  reliably with virtio video during installation.

## Scope

Tested on a single-node libvirt VM with Palette Enterprise 4.10.x. Multi-node
(`control_plane_count`) is not wired into the machine pool yet, and FIPS builds
are untested. The `appliance_vm` role is the only libvirt-specific piece;
replacing it leaves the rest reusable for bare metal or vSphere.

Not affiliated with Spectro Cloud. Palette and its artifacts are licensed
separately and are not distributed here; this repository only automates a
deployment you are already entitled to run. Consult the
[official documentation](https://docs.spectrocloud.com/enterprise-version/install-palette/palette-management-appliance/)
for supported installation procedures.

## License

Apache-2.0 — see [LICENSE](LICENSE).
