# artifacts/

Drop your Spectro Cloud downloads here. Everything in this directory is
gitignored — these files are licensed and large (the content bundle is ~10 GB).

Obtain them from [Artifact Studio](https://artifact-studio.spectrocloud.com),
which requires a customer login; ask your Spectro Cloud representative or open
a support ticket for access. Under **Install Palette Enterprise**, select your
version and show the artifacts, then download:

| File | Required |
|---|---|
| Content bundle, `*.tar.zst` | yes |
| Its detached signature, `*.tar.sig.bin` | recommended |
| Content-signing public key, `*.pem` | recommended |

The playbook finds them by extension, so exact filenames don't matter:

```
artifacts/
├── palette-enterprise-appliance-<version>.tar.zst
├── palette-enterprise-appliance-<version>.tar.sig.bin
└── spectro_public_key.pem
```

Verify a download before using it, per Spectro Cloud's bundle verification
instructions:

```bash
openssl dgst -sha256 -verify artifacts/spectro_public_key.pem \
  -signature artifacts/*.tar.sig.bin artifacts/*.tar.zst
# Verified OK
```

The playbook performs this same check before uploading anything, and fails the
run if it does not pass.
