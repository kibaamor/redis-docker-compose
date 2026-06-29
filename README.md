# Redis with Docker Compose

Two Redis deployment modes using Docker Compose:

- **Cluster mode** (`docker-compose.cluster.yaml`): 6-node Redis cluster (3 masters + 3 replicas) with automatic initialization
- **Single instance** (`docker-compose.single.yaml`): standalone Redis with Redis Commander web interface

## Features

- 6-node Redis cluster (3 master nodes + 3 replica nodes) — cluster mode
- Single Redis instance with Redis Commander — standalone mode
- Automatic cluster initialization — cluster mode
- Redis Commander web management interface
- Data persistence
- Health checks
- Resource limit configuration
- Log rotation configuration
- Configurable Redis bind address via `REDIS_BIND_ADDRESS`
- Configurable Redis Commander bind address via `REDIS_COMMANDER_BIND_ADDRESS`

## Quick Start

### Prerequisites

- Docker
- Docker Compose v2 (`docker compose`)

### Choose Deployment Mode

Use single instance mode when you want a simple local Redis server. Use cluster mode when you need to test Redis Cluster behavior such as sharding, replicas, and failover.

| Need | Recommended Mode |
| ---- | ---------------- |
| Local development, simple cache testing, or minimal resource usage | Single instance |
| Redis Cluster client testing, sharding behavior, replicas, or failover testing | Cluster |

Select the compose file that matches your needs, then symlink it to `docker-compose.yaml` so you can use `docker compose` without `-f` flags:

```bash
# For cluster mode (recommended if you need sharding and failover)
ln -sf docker-compose.cluster.yaml docker-compose.yaml

# For single instance mode (simpler, no clustering overhead)
ln -sf docker-compose.single.yaml docker-compose.yaml
```

> Note:
> `docker-compose.yaml` is gitignored, so each checkout can choose its own mode without changing tracked files.

### Configure

Copy the environment variable file if you want to customize the defaults:

```bash
cp .env.example .env
```

Then edit `.env` as needed.

> Tip:
> By default, Redis and Redis Commander bind to `127.0.0.1`. If you change bind addresses, see [Networking](#networking) before updating CLI commands or browser URLs.

### Start

```bash
docker compose up -d
```

### Verify Cluster Mode

```bash
# Check cluster initialization status
docker compose logs -f redis-cluster-init

# Check cluster state from node 1
docker compose exec redis-node-1 redis-cli -h 127.0.0.1 -p 7001 cluster info
```

> Note:
> On first startup, the `redis-cluster-init` service creates the cluster after all Redis nodes are healthy. If the cluster already exists, it detects that state and skips creation. Redis Commander starts after cluster initialization succeeds.

### Verify Single Instance Mode

```bash
# Check if Redis is running
docker compose logs -f redis

# Ping Redis
docker compose exec redis redis-cli -h 127.0.0.1 -p 6379 ping
```

### Open Redis Commander

With the default configuration, open <http://localhost:8081>.

> Tip:
> If you customize `REDIS_COMMANDER_BIND_ADDRESS` or `REDIS_COMMANDER_PORT`, open Redis Commander at the reachable host address and configured port. See [Networking](#networking) for bind address behavior.

### Stop

```bash
docker compose down
```

### Stop and Remove Data

```bash
docker compose down -v
```

## Common Commands

The commands below assume the default Redis bind address and ports:

- `REDIS_BIND_ADDRESS=127.0.0.1`
- `REDIS_PORT_1=7001` for cluster node 1
- `REDIS_PORT=6379` for single instance mode

> Tip:
> If you customize those values, replace the `-h` and `-p` arguments with reachable Redis address and port values. See [Networking](#networking) for bind address behavior.

### Cluster Mode

```bash
# Connect to node 1
docker compose exec redis-node-1 redis-cli -h 127.0.0.1 -p 7001

# View cluster info
docker compose exec redis-node-1 redis-cli -h 127.0.0.1 -p 7001 cluster info

# View cluster nodes
docker compose exec redis-node-1 redis-cli -h 127.0.0.1 -p 7001 cluster nodes

# Set key-value through cluster-aware mode
docker compose exec redis-node-1 redis-cli -c -h 127.0.0.1 -p 7001 set mykey "Hello Redis Cluster"

# Get key-value through cluster-aware mode
docker compose exec redis-node-1 redis-cli -c -h 127.0.0.1 -p 7001 get mykey

# Check which slot the key is in
docker compose exec redis-node-1 redis-cli -h 127.0.0.1 -p 7001 cluster keyslot mykey
```

### Single Instance Mode

```bash
# Connect to Redis
docker compose exec redis redis-cli -h 127.0.0.1 -p 6379

# Ping Redis
docker compose exec redis redis-cli -h 127.0.0.1 -p 6379 ping
```

### Monitoring

```bash
# View all service status
docker compose ps

# View service logs
docker compose logs -f

# View specific service logs (cluster mode)
docker compose logs -f redis-node-1

# View specific service logs (single instance mode)
docker compose logs -f redis

# View resource usage
docker stats
```

## Architecture

### Deployment Modes

| Mode | Compose File | Description |
| ---- | ------------ | ----------- |
| Cluster | `docker-compose.cluster.yaml` | 6-node Redis cluster with automatic initialization |
| Single | `docker-compose.single.yaml` | Standalone Redis instance with Redis Commander |

### Cluster Mode

Cluster mode runs 6 Redis nodes with `--cluster-replicas 1`:

- 3 master nodes
- 3 replica nodes, each master has 1 replica

Default Redis node ports:

| Service | Default Port | Environment Variable |
| ------- | ------------ | -------------------- |
| `redis-node-1` | `7001` | `REDIS_PORT_1` |
| `redis-node-2` | `7002` | `REDIS_PORT_2` |
| `redis-node-3` | `7003` | `REDIS_PORT_3` |
| `redis-node-4` | `7004` | `REDIS_PORT_4` |
| `redis-node-5` | `7005` | `REDIS_PORT_5` |
| `redis-node-6` | `7006` | `REDIS_PORT_6` |

### Single Instance Mode

Single instance mode runs one Redis service named `redis` on port `6379` by default. The port is configurable via `REDIS_PORT`.

### Networking

Both compose files use host networking mode. Redis binds directly to the host network interface configured by `REDIS_BIND_ADDRESS`, which defaults to `127.0.0.1`.

`REDIS_COMMANDER_BIND_ADDRESS` controls where the Redis Commander web UI listens. It does not control the Redis endpoints that Redis Commander connects to; those use `REDIS_BIND_ADDRESS`.

When Redis binds to `127.0.0.1`, use `127.0.0.1` in Redis CLI commands. When Redis binds to a LAN address, use that reachable host address. If Redis binds to `0.0.0.0`, do not use `0.0.0.0` as the client address; use `127.0.0.1` locally or the host IP remotely.

When Redis Commander binds to `0.0.0.0`, open it with `localhost` locally or the host IP remotely.

> Note:
> Host networking is most predictable on Linux. On Docker Desktop environments, host networking support and behavior can differ by platform and Docker Desktop version. If host networking is unavailable or behaves differently in your environment, these compose files may need port mappings instead of `network_mode: host`.

## Configuration

All configurations can be customized through the `.env` file. Refer to `.env.example` for the full list.

### Redis Configuration

| Environment Variable | Default Value | Scope | Description |
| --------- | ----- | ----- | ------ |
| `REDIS_VERSION` | `latest` | Both | Redis image version |
| `REDIS_BIND_ADDRESS` | `127.0.0.1` | Both | Network interface Redis binds to |
| `REDIS_PORT_1` | `7001` | Cluster | Port for Redis node 1 |
| `REDIS_PORT_2` | `7002` | Cluster | Port for Redis node 2 |
| `REDIS_PORT_3` | `7003` | Cluster | Port for Redis node 3 |
| `REDIS_PORT_4` | `7004` | Cluster | Port for Redis node 4 |
| `REDIS_PORT_5` | `7005` | Cluster | Port for Redis node 5 |
| `REDIS_PORT_6` | `7006` | Cluster | Port for Redis node 6 |
| `REDIS_PORT` | `6379` | Single | Port for Redis instance |
| `REDIS_APPENDONLY` | `no` | Both | Enable AOF persistence |
| `REDIS_APPENDFSYNC` | `everysec` | Both | AOF sync strategy (always/everysec/no) |
| `REDIS_SAVE` | `"900 1 300 10 60 10000"` | Both | RDB save strategy |
| `REDIS_MAXMEMORY` | `256mb` | Both | Maximum memory limit |
| `REDIS_MAXMEMORY_POLICY` | `allkeys-lru` | Both | Memory eviction policy |
| `REDIS_CPU_LIMIT` | `1.0` | Both | CPU limit |
| `REDIS_MEMORY_LIMIT` | `512M` | Both | Container memory limit |

### Redis Commander Configuration

| Environment Variable | Default Value | Scope | Description |
| --------- | ----- | ----- | ------ |
| `REDIS_COMMANDER_VERSION` | `latest` | Both | Redis Commander version |
| `REDIS_COMMANDER_BIND_ADDRESS` | `127.0.0.1` | Both | Network interface Redis Commander binds to |
| `REDIS_COMMANDER_PORT` | `8081` | Both | Web interface access port |
| `REDIS_COMMANDER_CPU_LIMIT` | `0.5` | Both | CPU limit |
| `REDIS_COMMANDER_MEMORY_LIMIT` | `256M` | Both | Memory limit |

### Logging Configuration

| Environment Variable | Default Value | Scope | Description |
| --------- | ----- | ----- | ------ |
| `MAX_LOG_FILE_SIZE` | `10m` | Both | Maximum size of a single log file |
| `MAX_LOG_FILE_COUNT` | `3` | Both | Number of log files to keep |

## Data Persistence

### Cluster Mode

Each Redis node has its own data volume:

- `redis_node_1_data`
- `redis_node_2_data`
- `redis_node_3_data`
- `redis_node_4_data`
- `redis_node_5_data`
- `redis_node_6_data`

### Single Instance Mode

Single Redis instance uses one data volume:

- `redis_data`

> Warning:
> Docker named volumes are preserved across container restarts. Redis data durability still depends on the configured Redis persistence settings, such as `REDIS_APPENDONLY` and `REDIS_SAVE`. Use `docker compose down -v` only when you want to remove the persisted data volumes.

## Failover

Cluster mode supports automatic failover:

- When a master node fails, the corresponding replica node will be automatically promoted to master
- Cluster node timeout is set to 5000ms
- Health checks are performed every 3 seconds

## Production Notes

These compose files are intended for local development, testing, and controlled environments.

- Keep `REDIS_BIND_ADDRESS=127.0.0.1` for local-only access.
- Avoid exposing Redis or Redis Commander to an untrusted network without access controls.
- If you bind Redis or Redis Commander to a LAN interface or `0.0.0.0`, restrict access with host firewall rules or a trusted network boundary.
- Pin `REDIS_VERSION` and `REDIS_COMMANDER_VERSION` instead of using `latest` when you need repeatable deployments.
- Review persistence settings before storing important data. `docker compose down -v` removes named volumes.

## Troubleshooting

### Cluster Creation Failed

```bash
# Check initialization logs
docker compose logs redis-cluster-init

# Check if all nodes are healthy
docker compose ps

# Reinitialize the cluster
docker compose restart redis-cluster-init
```

### Cannot Connect to Redis

```bash
# Cluster mode: check node logs
docker compose logs redis-node-1

# Cluster mode: ping node 1 with default bind address and port
docker compose exec redis-node-1 redis-cli -h 127.0.0.1 -p 7001 ping

# Single instance mode: check logs
docker compose logs redis

# Single instance mode: ping Redis with default bind address and port
docker compose exec redis redis-cli -h 127.0.0.1 -p 6379 ping
```

> Tip:
> If you changed `REDIS_BIND_ADDRESS`, use a reachable Redis address with `redis-cli -h`. See [Networking](#networking) for details.

### Cannot Connect to Redis Commander

```bash
# Check Redis Commander status
docker compose logs redis-commander

# Check Redis Commander service
docker compose ps redis-commander
```

> Tip:
> If you changed `REDIS_COMMANDER_BIND_ADDRESS` or `REDIS_COMMANDER_PORT`, confirm the reachable host address and configured port. See [Networking](#networking) for details.

### Node Cannot Start (Cluster Mode)

```bash
# Check node logs
docker compose logs redis-node-1
```

If resource limits are too low, edit `.env` and increase `REDIS_MEMORY_LIMIT` or `REDIS_CPU_LIMIT`.

## License

[MIT License](LICENSE)

## Contributing

Issues and pull requests are welcome.

Before submitting a change, verify the affected mode:

```bash
# Validate cluster compose configuration
docker compose -f docker-compose.cluster.yaml config

# Validate single instance compose configuration
docker compose -f docker-compose.single.yaml config
```

For README changes, check that documented service names, ports, and environment variables still match the compose files and `.env.example`.
