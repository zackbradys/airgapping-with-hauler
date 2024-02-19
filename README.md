---
title: Simplifying the Airgap Experience with Rancher Government Hauler
author: Zack Brady - Field Engineer
contact: zack.brady@ranchergovernment.com
---

![rgs-hauler-banner](images/rgs-hauler-banner.png)

# Simplifying the Airgap Experience with Rancher Government Hauler

### Table of Contents

- [Introduction](#introduction)
- [Understanding the Airgap](#understanding-the-airgap)
- [Challenges with the Airgap](#challenges-with-the-airgap)
- [Rancher Government Hauler](#rancher-government-hauler)
- [Airgaping with Hauler](#cloud-native-options)
- [Bootstrapping Airgapped Environments](#bootstrapping-airgapped-environments)
- [What is Next?](#what-is-next)

## Introduction

Text here...

## Understanding the Airgap

I always like to start by level setting so let's start off with the basis or an airgap or disconnected environment. An airgap is a highly secure environment that is physically isolated from external connectivity (usually the internet), ensuring complete data isolation and preventing unauthorized access or data transfer.

- **Government and Military:** These airgaps protect sensitive government and military information and critical defense infrastructure, ensuring that classified data remains secure and inaccessible to unauthorized entities.
- **Critical Infrastructure:** These airgaps secure power grids, water supply systems, and transportation networks from cyber threats, shielding them from potential disruptions and ensuring their continuous operation.
- **Financial Institutions:** These airgaps safeguard financial transactions and customer data from cyberattacks, maintaining the trust and integrity of financial systems.
- **Research Facilities:** These airgaps protect valuable research data, intellectual property, and proprietary information, preserving the integrity and confidentiality of research efforts.

## Challenges with the Airgap

Now that we understand airgapped or disconnected environments, let's take a look at some of the challenges and complexitites with them:

- **Data Transfers:** Transferring data in and out of an airgapped system is complex due to the lack of network connectivity, often necessitating the use of physical media, trusted intermediaries, and meticulous approval processes to ensure data security.
- **Usability vs. Security Balance:** Balancing security with usability in airgapped environments can be intricate, as the limited convenience poses challenges for user interactions and system operations.
- **Maintenance Complexity:** Regular system updates, package and dependency updates, and overall maintenance can be challenging in airgapped systems, as the limited network connectivity complicates the process, requiring meticulous planning and execution.
- **Costly Implementation:** Implementing and maintaining airgapped systems can be costly, demanding specialized workflows, protocols, and security measures to uphold the system's integrity and security.

## Rancher Government Hauler

**Rancher Government Hauler** simplifies the airgap experience without requiring the adoption of a specific workflow. Assets such as container images, helm charts, and files are represented as types of artifacts within Hauler Collections and Hauler Content. Operators can easily fetch, store, package, validate, and distribute these assets and artifacts with declarative manifests or through the command line.

**Rancher Government Hauler** achieves this by storing as artifacts within contents and collections as OCI Compliant Artifacts. Allowing operators to easily manage them and serve them as an embedded registry and fileserver, thereby reducing the dependencies for bootstrapping airgapped environments.

Text here...

![rgs-hauler-banner](images/hauler-diagram.png)

Text here...

![rgs-hauler-banner](images/hauler-workflow-diagram.png)

## Airgapping with Hauler

Now that we understand the basics of Hauler and the basis of airgapped environments. Let's dive into installing and using it!

Text here...

```bash
# install latest release
[root@ip-172-31-32-174 hauler]# curl -sfL https://get.hauler.dev | bash

[INFO] Hauler: Starting Installation...
- Version: v1.0.0
- Platform: linux
- Architecture: amd64

[INFO] Hauler: Starting Checksum Verification...
- Expected Checksum: 966fc0fedbc445447f261ecfd03e145b615e764303f7ab685a5e42300beae36b
- Determined Checksum: 966fc0fedbc445447f261ecfd03e145b615e764303f7ab685a5e42300beae36b
- Successfully Verified Checksum: hauler_1.0.0_linux_amd64.tar.gz

[INFO] Hauler: Successfully Installed at /usr/local/bin/hauler
- Hauler v1.0.0 is now available for use!
- Documentation: https://hauler.dev
```

Text here...

```bash
# add a image... defaults to latest and docker.io
[root@ip-172-31-32-174 hauler]# hauler store add image neuvector/scanner
3:31PM INF added 'image' to store at [index.docker.io/neuvector/scanner:latest]

# add a image with a specific platform and with supply chain artifacts
# may not work for all users due to the specified registry
[root@ip-172-31-32-174 hauler]# hauler store add image rgcrprod.azurecr.us/longhornio/longhorn-ui:v1.6.0 --platform linux/amd64 --key carbide-key.pub
3:32PM INF signature verified for image [rgcrprod.azurecr.us/longhornio/longhorn-ui:v1.6.0]
3:32PM INF added 'image' to store at [rgcrprod.azurecr.us/longhornio/longhorn-ui:v1.6.0]

# add a helm chart with a specific version
[root@ip-172-31-32-174 hauler]# hauler store add chart rancher --repo https://releases.rancher.com/server-charts/stable --version 2.8.2
3:33PM INF added 'chart' to store at [hauler/rancher:2.8.2], with digest [sha256:27e742f51e66e32512509a95523bc9a531ec63f723c730b47685e7678cbc30d3]

# add a file and assign it a new name
[root@ip-172-31-32-174 hauler]# hauler store add file https://get.rke2.io --name install.sh
3:34PM INF added 'file' to store at [hauler/get.rke2.io:latest], with digest [sha256:0bffbf5fecf9dda70c112153a6ea90392cc2e67c55fa82cc7bb0679b03ef68e0]
```

Text here...

```bash
cat << EOF >> hauler-manifest.yaml
apiVersion: content.hauler.cattle.io/v1alpha1
kind: Images
metadata:
  name: hauler-content-images-example
spec:
  images:
    - name: neuvector/scanner:latest
    - name: rgcrprod.azurecr.us/longhornio/longhorn-ui:v1.6.0
      key: carbide-key.pub
      platform: linux/amd64
---
apiVersion: content.hauler.cattle.io/v1alpha1
kind: Charts
metadata:
  name: hauler-content-charts-example
spec:
  charts:
    - name: rancher
      repoURL: https://releases.rancher.com/server-charts/stable
      version: 2.8.2
---
apiVersion: content.hauler.cattle.io/v1alpha1
kind: Files
metadata:
  name: hauler-content-files-example
spec:
  files:
    - path: https://get.rke2.io
      name: install.sh
EOF

[root@ip-172-31-32-174 hauler]# hauler store sync --files hauler-manifest.yaml
3:36PM INF syncing [content.hauler.cattle.io/v1alpha1, Kind=Images] to store
3:36PM INF added 'image' to store at [index.docker.io/neuvector/scanner:latest]
3:36PM INF signature verified for image [rgcrprod.azurecr.us/longhornio/longhorn-ui:v1.6.0]
3:36PM INF added 'image' to store at [rgcrprod.azurecr.us/longhornio/longhorn-ui:v1.6.0]
3:36PM INF syncing [content.hauler.cattle.io/v1alpha1, Kind=Charts] to store
3:36PM INF added 'chart' to store at [hauler/rancher:2.8.2], with digest [sha256:27e742f51e66e32512509a95523bc9a531ec63f723c730b47685e7678cbc30d3]
3:36PM INF syncing [content.hauler.cattle.io/v1alpha1, Kind=Files] to store
3:36PM INF added 'file' to store at [hauler/install.sh:latest], with digest [sha256:0a6317871b81a2fb0afe1369057aa69209c4668f9d359e38e79e7817f2a10107]
```

Text here...

```bash
# view the content in the local hauler store
[root@ip-172-31-32-174 hauler]# hauler store info
+-------------------------------+-------+-------------+----------+----------+
| REFERENCE                     | TYPE  | PLATFORM    | # LAYERS | SIZE     |
+-------------------------------+-------+-------------+----------+----------+
| hauler/install.sh:latest      | file  | -           |        1 | 24.7 kB  |
| hauler/rancher:2.8.2          | chart | -           |        1 | 15.0 kB  |
| longhornio/longhorn-ui:v1.6.0 | image | linux/amd64 |        6 | 73.2 MB  |
|                               | atts  | -           |        1 | 11.6 kB  |
|                               | sbom  | -           |        1 | 2.5 MB   |
|                               | sigs  | -           |        1 | 258 B    |
| neuvector/scanner:latest      | image | linux/amd64 |        4 | 174.3 MB |
|                               | image | linux/arm64 |        4 | 163.6 MB |
+-------------------------------+-------+-------------+----------+----------+
|                                                        TOTAL   | 465.7 MB |
+-------------------------------+-------+-------------+----------+----------+
```

Text here...

```bash
# save and export the content in the local hauler store
[root@ip-172-31-32-174 hauler]# hauler store save --filename haul.tar.zst
3:40PM INF saved store [store] -> [/opt/hauler/haul.tar.zst]
```

Text here...

```bash
# load and import the airgapped content to the new local hauler store
[root@ip-172-31-32-174 hauler]# hauler store load haul.tar.zst
3:43PM INF loading content from [haul.tar.zst] to [store]
```

Text here...

```bash
# view the content in the local hauler store
[root@ip-172-31-32-174 hauler]# hauler store info
+-------------------------------+-------+-------------+----------+----------+
| REFERENCE                     | TYPE  | PLATFORM    | # LAYERS | SIZE     |
+-------------------------------+-------+-------------+----------+----------+
| hauler/install.sh:latest      | file  | -           |        1 | 24.7 kB  |
| hauler/rancher:2.8.2          | chart | -           |        1 | 15.0 kB  |
| longhornio/longhorn-ui:v1.6.0 | image | linux/amd64 |        6 | 73.2 MB  |
|                               | atts  | -           |        1 | 11.6 kB  |
|                               | sbom  | -           |        1 | 2.5 MB   |
|                               | sigs  | -           |        1 | 258 B    |
| neuvector/scanner:latest      | image | linux/amd64 |        4 | 174.3 MB |
|                               | image | linux/arm64 |        4 | 163.6 MB |
+-------------------------------+-------+-------------+----------+----------+
|                                                        TOTAL   | 465.7 MB |
+-------------------------------+-------+-------------+----------+----------+
```

## Bootstrapping Airgap Environments

Text here...

```bash
# serve the content as a registry from the hauler store
# defaults to <FQDN or IP>:5000
[root@ip-172-31-32-174 hauler]# hauler store serve registry
3:49PM INF neuvector/scanner:latest
3:49PM INF longhornio/longhorn-ui:v1.6.0
3:49PM INF hauler/install.sh:latest
3:49PM INF hauler/rancher:2.8.2
3:49PM INF copied artifacts to [127.0.0.1:43273]
3:49PM INF starting registry on port [5000]

WARN[0003] No HTTP secret provided - generated random secret. This may cause problems with uploads if multiple registries are behind a load-balancer. To provide a shared secret, fill in http.secret in the configuration file or set the REGISTRY_HTTP_SECRET environment variable.  go.version=go1.21.7 version=v3.0.0+unknown
INFO[0003] redis not configured                          go.version=go1.21.7 version=v3.0.0+unknown
INFO[0003] Starting upload purge in 38m0s                go.version=go1.21.7 version=v3.0.0+unknown
INFO[0003] using inmemory blob descriptor cache          go.version=go1.21.7 version=v3.0.0+unknown
INFO[0003] listening on [::]:5000                        go.version=go1.21.7 version=v3.0.0+unknown

# view the registry and verify the available images
[root@ip-172-31-32-174 hauler]# curl -sfL localhost:5000/v2/_catalog | jq
{
  "repositories": [
    "hauler/install.sh",
    "hauler/rancher",
    "longhornio/longhorn-ui",
    "neuvector/scanner"
  ]
}

# serve the file content as a fileserver from the hauler store
# defaults to <FQDN or IP>:8080
[root@ip-172-31-32-174 hauler]# hauler store serve fileserver
3:52PM INF copied artifacts to [store-files]
3:52PM INF starting file server on port [8080]

# view the fileserver and verify the available files
[root@ip-172-31-32-174 hauler]# curl -sfL http://localhost:8080
<pre>
<a href="install.sh">install.sh</a>
<a href="rancher-2.8.2.tgz">rancher-2.8.2.tgz</a>
</pre>
```
### Copying vs Bootstrapping

Text here...

```bash
# copy the content to a registry from the hauler store
# copies oci compliant artifacts
[root@ip-172-31-32-174 hauler]# hauler store copy registry://localhost:5000
3:55PM INF hauler/install.sh:latest
3:55PM INF longhornio/longhorn-ui:v1.6.0
3:55PM INF hauler/rancher:2.8.2
3:55PM INF neuvector/scanner:latest
3:55PM INF longhornio/longhorn-ui:v1.6.0
3:55PM INF copied artifacts to [localhost:5000]

# copy the content to a directory from the hauler store
# copies non oci compliant artifacts
[root@ip-172-31-32-174 hauler]# hauler store copy dir://hauler-files
3:57PM INF copied artifacts to [hauler-files]
```


## What is Next?
