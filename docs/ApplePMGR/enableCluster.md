Sets the power target of one SoC cluster, the only writer of `CLUSTER_CTL`
- `enableCluster(u8 clusterID, bool enable, bool wait, u32 die)`, `0xfffffe00098957e4`, vtable `0xbd0`, PAC disc `0x8f30`
- Register: `CLUSTER_CTL` + 4 * [[getPhysicalClusterID]](clusterID)
	1. Off and cluster has `soc-clusters` flag bit 2 and `rail & 0xFE != 4`: [[enableTVM]] first (AVE yes, ANE no)
	2. `(reg & ~0xF) | (enable ? 0xF : 0)`, keeps bit 5 (writes the interrupt status back as 1)
	3. `wait`: until bit 4 clears, then caches `reg & 0xF` as the cluster state (`clusterInfo + 8`)
	4. On: TVM back on after the wait (same condition)
	5. `IOStateReporter` bookkeeping ("SCL..." / "ACT" / "INACT"), nothing else
- Callers: `_syncDeviceStatus` (boot / S2R wake), [[_syncDeviceStatusChange]] (on), `_clusterHysteresisTimeoutHandlerGated` (off), `quiesceHW`, `configCPUComplexPowerState`
- Same write smcpg does, every Linux cut still hangs -> [[Cluster gating]]
