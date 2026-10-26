Cluster ID (`soc-clusters` byte 3) -> physical ID (byte 0) = `CLUSTER_CTL` word index
- Via `physClusters[0][clusterID]` -> `clusterInfo[index].desc` byte 0, 255 if the cluster doesn't exist
