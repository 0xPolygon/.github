Hi Everyone,

We have released bor [v2.10.2 stable](https://github.com/0xPolygon/bor/releases/tag/v2.10.2) for Polygon Mainnet.

v2.10.2 comes with two main features, pipelined state root computation for block import and WIT2 BP-signed witness announcements. Pipelined state root computation is disabled by default — if you would like to enable it, see the "Enabling pipelined state root computation" section below. WIT2 needs no flag and negotiates per connection, so mixed WIT1/WIT2 meshes work without a coordinated upgrade.

This release is not a hardfork — there are no fork activation heights, chain config or genesis changes, no database schema change, and no resync is required.

## Steps to upgrade bor

**Stop bor service**

```bash
sudo service bor stop
```

**Install Bor with a version tag, network name (mainnet), and node type (sentry, validator, or archive)**

```bash
# Replace the node type
curl -L https://raw.githubusercontent.com/0xPolygon/install/main/bor.sh | bash -s -- v2.10.2 <network> <node_type>
```

**Check bor version**

```bash
/usr/bin/bor version
# It should print
# Version: 2.10.2
```

**Restart bor service**

```bash
sudo service bor start
```

## Enabling pipelined state root computation (optional)

This feature is off by default. To turn it on, either pass the CLI flag:

```bash
bor server --pipeline.enable-import-src
```

or set it in `config.toml`:

```toml
[pipeline]
  enable-import-src = true
```

The feature targets block import and sync, not block production.

Validators running Clef must whitelist the new mimetype `application/x-bor-wit2-announce` for WIT2, otherwise the producer falls back to unsigned WIT1 announcements.

## Release notes

This release contains new features and bug fixes on top of v2.10.1.

### Highlights

**Pipelined state root computation for block import - [#2180](https://github.com/0xPolygon/bor/pull/2180)**

Overlaps the state-root computation of block N with transaction execution of block N+1 during block import, improving sync and catch-up throughput. Targets import and sync, not block production. This feature is disabled by default. To enable it, either pass the CLI flag `bor server --pipeline.enable-import-src` or set `enable-import-src = true` under `[pipeline]` in `config.toml`.

**WIT2: BP-signed witness announcements - [#2208](https://github.com/0xPolygon/bor/pull/2208)**

Block producers now sign a commitment over each witness, so peers can verify and relay the announcement without executing the block first, and can serve the witness body before importing it. This cuts witness-propagation latency for stateless validators several hops from a producer.

No flag or coordinated upgrade needed - the wit protocol negotiates per connection and mixed WIT1/WIT2 meshes work. A signed announcement isn't relayed past the first WIT1 hop, so upgrade each validator together with its sentries. Validators running Clef must whitelist the new mimetype `application/x-bor-wit2-announce`, or the producer falls back to unsigned WIT1 announcements.

### Other changes

**Fixes**

- eth, eth/fetcher, eth/protocols/eth: retry dangling tx-fetch peers instead of excluding them forever - fixes sentries sitting with a near-empty mempool for hours after a restart (POS-3689) by @marcello33 in [#2367](https://github.com/0xPolygon/bor/pull/2367)
- consensus/bor: don't leak the live tracer into non-import system transactions - a node-wide live tracer (--vmtrace) was being fired from RPC goroutines via eth_simulateV1 and debug_trace* state regeneration by @nebojsa94 in [#2353](https://github.com/0xPolygon/bor/pull/2353)
- core/blockstm, core: exclude base-state-delegated senders from the BlockSTM v2 nonce pre-compute, where V2 and the serial processor could disagree; harden the witness parity harness by @cffls in [#2383](https://github.com/0xPolygon/bor/pull/2383)
- core, miner: fix nightly pipeline race failures by @pratikspatil024 in [#2371](https://github.com/0xPolygon/bor/pull/2371)

**CI, build, and tests**

- Remove support for publishing images to Docker Hub by @0xsajal in [#2372](https://github.com/0xPolygon/bor/pull/2372)
- ci: add a pipeline-enabled kurtosis e2e leg (POS-3697) by @pratikspatil024 in [#2368](https://github.com/0xPolygon/bor/pull/2368)
- ci: run kurtosis E2E safely on fork PRs by @pratikspatil024 in [#2380](https://github.com/0xPolygon/bor/pull/2380)
- ci: bump kurtosis-pos pin and Actions versions in kurtosis workflows by @kamuikatsurgi in [#2370](https://github.com/0xPolygon/bor/pull/2370)
- ci: bump kurtosis-pos pin to v1.4.2 by @marcello33 in [#2377](https://github.com/0xPolygon/bor/pull/2377)
- tests/bor: harden test peering against simultaneous-connect collisions by @pratikspatil024 in [#2369](https://github.com/0xPolygon/bor/pull/2369)
- build: increase race-test package timeout by @pratikspatil024 in [#2378](https://github.com/0xPolygon/bor/pull/2378)
- eth: avoid a race in relay-fetch test setup by @pratikspatil024 in [#2387](https://github.com/0xPolygon/bor/pull/2387)
- chore(ci): upgrade to codegenie@v0.5.6 by @0xPolygonBot in [#2379](https://github.com/0xPolygon/bor/pull/2379)
- v2.10.1 back-merge to master by @pratikspatil024 in [#2390](https://github.com/0xPolygon/bor/pull/2390)

**Dependencies**

- Bump github.com/pion/dtls/v3 from 3.0.11 to 3.1.4 by @dependabot in [#2338](https://github.com/0xPolygon/bor/pull/2338)
- Bump github.com/pion/stun/v3 from 3.1.1 to 3.1.5 by @dependabot in [#2339](https://github.com/0xPolygon/bor/pull/2339)
- Bump github.com/rabbitmq/amqp091-go from 1.10.0 to 1.13.0 by @dependabot in [#2384](https://github.com/0xPolygon/bor/pull/2384)
- Bump google.golang.org/grpc from 1.79.3 to 1.83.1 by @dependabot in [#2381](https://github.com/0xPolygon/bor/pull/2381)

### Upgrade notes

No hardfork, no resync, backwards compatible. [#2180](https://github.com/0xPolygon/bor/pull/2180) is off by default and [#2208](https://github.com/0xPolygon/bor/pull/2208) negotiates per connection with WIT1 peers.

Docker Hub images are gone. Image publishing to Docker Hub was removed in [#2372](https://github.com/0xPolygon/bor/pull/2372) ahead of the 1 September 2026 sunset - pull from GHCR instead. See the [GHCR announcement](https://forum.polygon.technology/t/announcement-ghcr-images-and-docker-sunset/22022).

### New Contributors

- @0xPolygonBot made their first contribution in [#2379](https://github.com/0xPolygon/bor/pull/2379)
- @nebojsa94 made their first contribution in [#2353](https://github.com/0xPolygon/bor/pull/2353)

### What's Changed

Full changelog: [Comparing v2.10.1...v2.10.2](https://github.com/0xPolygon/bor/compare/v2.10.1...v2.10.2)

### Docker Images

Docker Hub is no longer used. Image publishing to Docker Hub was removed ahead of its 1 September 2026 sunset. Pull images from GitHub Container Registry (GHCR) instead:

```bash
docker pull ghcr.io/0xpolygon/bor:v2.10.2
```

Bor images: [GitHub Container Registry](https://github.com/0xPolygon/bor/pkgs/container/bor)

Images are published for linux/amd64 and linux/arm64 with build provenance attestation.

Thanks,
Polygon Team
