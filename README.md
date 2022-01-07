# shared-ksops-secret-demo

This repository contains a demo of sharing a KSOPS encyrpted secret between 
multiple namespaces.  Please note that this only works within a cluster since 
a decryption key should not be shared between cluster.

## Folder Organization

```
.
├── apps                                 (Application configuration common to all clusters)
│   ├── my-app
│   │   ├── deployment.yaml
│   │   ├── kustomization.yaml
│   │   └── secret.placeholder.yaml      (This file isn't actually included in kustomization; it is only here for reference)
│   └── my-other-app
│       ├── deployment.yaml
│       ├── kustomization.yaml
│       └── secret.placeholder.yaml      (This file isn't actually included in kustomization; it is only here for reference)
└── clusters
    └── eks-01
        ├── apps                         (Overrides for apps on this cluster)
        │   ├── kustomization.yaml
        │   ├── my-app
        │   │   └── kustomization.yaml
        │   └── my-other-app
        │       └── kustomization.yaml
        ├── shared                       (Configuration shared across multiple resources on this cluster)
        │   ├── kustomization.yaml
        │   ├── secret.enc.yaml
        │   └── secret.ksops.yaml
        └── .sops.yaml                   (Cluster-wide sops configuration)
```

The kustomization in the cluster-specific app configuration loads the general app configuration, then loads the shared cluster-specific configuration.  E.g.,
`./clusters/eks-01/apps/my-app` includes `./apps/my-app` then `./clusters/eks-01/shared`.  In a real-world scenario, it would probably include cluster and app-specific overrides too.

## Demo

The following instructions assume you are operating in a Linux or Linux-like environment

### Install ksops and kustomize

If not already installed

#### Install ksops

For detailed instructions see [https://github.com/viaduct-ai/kustomize-sops#installation].

```
export XDG_CONFIG_HOME=~/.config
source <(curl -s https://raw.githubusercontent.com/viaduct-ai/kustomize-sops/master/scripts/install-ksops-archive.sh)
```

#### Install kustomize

See [https://kubectl.docs.kubernetes.io/installation/kustomize/binaries/].

### Generate a PGP key for testing

`gpg --generate-key`

```
njlaw in ncc-1701-h in ☸ cco-net2-eks-01 (cco-net2-backend-dev3) in shared-sops-secret-demo on  master [?] on ☁️  (us-east-1) took 1m14s 
❯ gpg --generate-key 
gpg (GnuPG) 2.2.19; Copyright (C) 2019 Free Software Foundation, Inc.

GnuPG needs to construct a user ID to identify your key.

Real name: Test Local SOPS Key
Email address: 
You selected this USER-ID:
    "Test Local SOPS Key"

Change (N)ame, (E)mail, or (O)kay/(Q)uit? O

gpg: key 0E0A820CCF37909B marked as ultimately trusted
gpg: revocation certificate stored as '/home/njlaw/.gnupg/openpgp-revocs.d/140BDC9A5872DD4AE13B7E3A0E0A820CCF37909B.rev'
public and secret key created and signed.

pub   rsa3072 2022-01-07 [SC] [expires: 2024-01-07]
      140BDC9A5872DD4AE13B7E3A0E0A820CCF37909B
uid                      Test Local SOPS Key
sub   rsa3072 2022-01-07 [E] [expires: 2024-01-07]
```

### Set the public key ID output by gpg in the SOPS configuration

Edit `./clusters/eks-01/.sops.yaml` and replace the `pgp` field with your key ID.

### Verify output

Run `kustomize build --enable-alpha-plugins ./clusters/eks-01/apps/my-app`
and `kustomize build --enable-alpha-plugins ./clusters/eks-01/apps/my-other-app`

Both outputs will include the shared secret, but in different namespaces.
