Sets the power target of one SoC cluster
- `enableCluster(u8 clusterID, bool enable, bool wait, u32 die)`, mangled `__ZN9ApplePMGR13enableClusterEhbbj`
- Register: `CLUSTER_CTL` + 4 * [[getPhysicalClusterID]](clusterID)
	- `(reg & ~0xF) | (enable ? 0xF : 0)`, bits 3:0 = target
	- `wait`: waits for bit 4 to clear (busy?), then caches the state via `getClusterPowerState`
- Clusters with flag bit 2 (`soc-clusters` byte 2) switch TVM for their voltage rail
- Rest is `IOStateReporter` bookkeeping ("SCL..." / "ACT" / "INACT")
