---
title: Restoring from Backup
---

import Tabs from '@theme/Tabs';
import TabItem from '@theme/TabItem';

The details of restoring your cluster from backup are different depending on your version of RKE.

<Tabs>
<TabItem value="RKE v0.2.0+">

If there is a disaster with your Kubernetes cluster, you can use `rke etcd snapshot-restore` to recover your etcd. This command reverts etcd to a specific snapshot and should be run on an etcd node of the the specific cluster that has suffered the disaster.

The following actions will be performed when you run the command:

- Syncs the snapshot or downloads the snapshot from S3, if necessary.
- Checks snapshot checksum across etcd nodes to make sure they are identical.
- Deletes your current cluster and cleans old data by running `rke remove`. This removes the entire Kubernetes cluster, not just the etcd cluster.
- Rebuilds the etcd cluster from the chosen snapshot.
- Creates a new cluster by running `rke up`.
- Restarts cluster system pods.

:::danger

You should back up any important data in your cluster before running `rke etcd snapshot-restore` because the command deletes your current Kubernetes cluster and replaces it with a new one.

:::

The snapshot used to restore your etcd cluster can either be stored locally in `/opt/rke/etcd-snapshots` or from a S3 compatible backend.

_Available as of v1.1.4_

If the snapshot contains the cluster state file, it will automatically be extracted and used for the restore. If you want to force the use of the local state file, you can add `--use-local-state` to the command. If the snapshot was created using an RKE version before v1.1.4, or if the snapshot does not contain a state file, make sure the cluster state file (by default available as `cluster.rkestate`) is present before executing the command.

### Example of Restoring from a Local Snapshot

To restore etcd from a local snapshot, run:

```
$ rke etcd snapshot-restore --config cluster.yml --name mysnapshot
```

The snapshot is assumed to be located in `/opt/rke/etcd-snapshots`.

**Note:** The `pki.bundle.tar.gz` file is not needed because RKE v0.2.0 changed how the [Kubernetes cluster state is stored](../../installation/installation.md#kubernetes-cluster-state).

### Example of Restoring from a Snapshot in S3

When restoring etcd from a snapshot located in S3, the command needs the S3 information in order to connect to the S3 backend and retrieve the snapshot.

```shell
$ rke etcd snapshot-restore \
--config cluster.yml \
--name snapshot-name \
--s3 \
--access-key S3_ACCESS_KEY \
--secret-key S3_SECRET_KEY \
--bucket-name s3-bucket-name \
--folder s3-folder-name \ # Optional - Available as of v0.3.0
--s3-endpoint s3.amazonaws.com
```
**Note:** if you were restoring a cluster that had Rancher installed, the Rancher UI should start up after a few minutes; you don't need to re-run Helm.

### Options for `rke etcd snapshot-restore`

| Option | Description | S3 Specific |
| --- | --- | ---|
| `--name` value            |  Specify snapshot name | |
| `--config` value          |  Specify an alternate cluster YAML file (default: `cluster.yml`) [$RKE_CONFIG] | |
| `--use-local-state`       | Force the use of the local state file instead of looking for a state file in the snapshot _Available as of v1.1.4_ | |
| `--s3`                    |  Enabled backup to s3 |* |
| `--s3-endpoint` value     |  Specify s3 endpoint url (default: "s3.amazonaws.com") | * |
| `--access-key` value      |  Specify s3 accessKey | *|
| `--secret-key` value      |  Specify s3 secretKey | *|
| `--bucket-name` value     |  Specify s3 bucket name | *|
| `--folder` value     |   Specify folder inside  bucket where backup will be stored. This is optional.  This is optional. _Available as of v0.3.0_ | *|
| `--region` value          |  Specify the s3 bucket location (optional) | *|
| `--ssh-agent-auth`      |   [Use SSH Agent Auth defined by SSH_AUTH_SOCK](../../config-options/config-options.md#ssh-agent) | |
| `--ignore-docker-version`  | [Disable Docker version check](../../config-options/config-options.md#supported-docker-versions) |

</TabItem>
<TabItem value="RKE before v0.2.0">

If there is a disaster with your Kubernetes cluster, you can use `rke etcd snapshot-restore` to recover your etcd. This command reverts etcd to a specific snapshot and should be run on an etcd node of the the specific cluster that has suffered the disaster.

The following actions will be performed when you run the command:

- Removes the old etcd cluster
- Rebuilds the etcd cluster using the local snapshot

Before you run this command, you must:

- Run `rke remove` to remove your Kubernetes cluster and clean the nodes
- Download your etcd snapshot from S3, if applicable. Place the etcd snapshot and the `pki.bundle.tar.gz` file in `/opt/rke/etcd-snapshots`. Manually sync the snapshot across all `etcd` nodes.

After the restore, you must rebuild your Kubernetes cluster with `rke up`.

:::danger

You should back up any important data in your cluster before running `rke etcd snapshot-restore` because the command deletes your current etcd cluster and replaces it with a new one.

:::

### Example of Restoring from a Local Snapshot

To restore etcd from a local snapshot, run:

```
$ rke etcd snapshot-restore --config cluster.yml --name mysnapshot
```

The snapshot is assumed to be located in `/opt/rke/etcd-snapshots`.

The snapshot must be manually synched across all `etcd` nodes.

The `pki.bundle.tar.gz` file is also expected to be in the same location.

### Options for `rke etcd snapshot-restore`

| Option | Description |
| --- | --- |
| `--name` value            |  Specify snapshot name |
| `--config` value          |  Specify an alternate cluster YAML file (default: `cluster.yml`) [$RKE_CONFIG] |
| `--ssh-agent-auth`      |   [Use SSH Agent Auth defined by SSH_AUTH_SOCK](../../config-options/config-options.md#ssh-agent) |
| `--ignore-docker-version`  | [Disable Docker version check](../../config-options/config-options.md#supported-docker-versions) |

</TabItem>
</Tabs>
