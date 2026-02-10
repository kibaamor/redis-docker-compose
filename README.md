# Redis Cluster with Docker Compose

A Redis cluster deployment using Docker Compose, including 6 Redis nodes (3 masters + 3 replicas) and Redis Commander web interface.

## Features

- ✅ 6-node Redis cluster (3 master nodes + 3 replica nodes)
- ✅ Automatic cluster initialization
- ✅ Redis Commander web management interface
- ✅ Data persistence
- ✅ Health checks
- ✅ Resource limit configuration
- ✅ Log rotation configuration

## Architecture

### Redis Nodes

- **redis-node-1**: port 7001 (configurable via `REDIS_PORT_1`)
- **redis-node-2**: port 7002 (configurable via `REDIS_PORT_2`)
- **redis-node-3**: port 7003 (configurable via `REDIS_PORT_3`)
- **redis-node-4**: port 7004 (configurable via `REDIS_PORT_4`)
- **redis-node-5**: port 7005 (configurable via `REDIS_PORT_5`)
- **redis-node-6**: port 7006 (configurable via `REDIS_PORT_6`)

**Note**: This setup uses host networking mode, so Redis nodes bind directly to the host's network interface. Default ports are 7001-7006, but can be customized via environment variables.

### Cluster Topology

The cluster uses `--cluster-replicas 1` configuration, which means:

- 3 master nodes
- 3 replica nodes, each master has 1 replica

### Redis Commander

- Web management interface, default port: `18081`
- Visual management of all cluster nodes
- Cluster mode support

## Quick Start

### Prerequisites

- Docker
- Docker Compose

### Start the Cluster

1. Copy the environment variable file (optional): `cp .env.example .env`
2. Modify the configuration in the `.env` file as needed
3. Start the services: `docker compose up -d`
4. Check cluster status: `docker compose logs -f cluster-init`
5. Access Redis Commander: <http://localhost:18081>

### Stop the Cluster

```bash
docker compose down
```

### Stop the Cluster and Remove Data

```bash
docker compose down -v
```

## Configuration

All configurations can be customized through the `.env` file. Refer to the `.env.example` file.

### Redis Configuration

| Environment Variable | Default Value | Description |
| --------- | ----- | ------ |
| `REDIS_VERSION` | `latest` | Redis image version |
| `REDIS_BIND_ADDRESS` | `127.0.0.1` | Network interface to bind to |
| `REDIS_PORT_1` | `7001` | Port for Redis node 1 |
| `REDIS_PORT_2` | `7002` | Port for Redis node 2 |
| `REDIS_PORT_3` | `7003` | Port for Redis node 3 |
| `REDIS_PORT_4` | `7004` | Port for Redis node 4 |
| `REDIS_PORT_5` | `7005` | Port for Redis node 5 |
| `REDIS_PORT_6` | `7006` | Port for Redis node 6 |
| `REDIS_APPENDONLY` | `no` | Enable AOF persistence |
| `REDIS_APPENDFSYNC` | `everysec` | AOF sync strategy (always/everysec/no) |
| `REDIS_SAVE` | `"900 1 300 10 60 10000"` | RDB save strategy |
| `REDIS_MAXMEMORY` | `256mb` | Maximum memory limit |
| `REDIS_MAXMEMORY_POLICY` | `allkeys-lru` | Memory eviction policy |
| `REDIS_CPU_LIMIT` | `1.0` | CPU limit |
| `REDIS_MEMORY_LIMIT` | `512M` | Container memory limit |

### Redis Commander Configuration

| Environment Variable | Default Value | Description |
| --------- | ----- | ------ |
| `REDIS_COMMANDER_VERSION` | `latest` | Redis Commander version |
| `REDIS_COMMANDER_BIND_ADDRESS` | `127.0.0.1` | Network interface to bind to |
| `REDIS_COMMANDER_PORT` | `18081` | Web interface access port |
| `REDIS_COMMANDER_CPU_LIMIT` | `0.5` | CPU limit |
| `REDIS_COMMANDER_MEMORY_LIMIT` | `256M` | Memory limit |

### Logging Configuration

| Environment Variable | Default Value | Description |
| --------- | ----- | ------ |
| `MAX_LOG_FILE_SIZE` | `10m` | Maximum size of a single log file |
| `MAX_LOG_FILE_COUNT` | `3` | Number of log files to keep |

## Common Commands

### Connect to Redis Nodes

```bash
# Connect to node 1
docker exec -it redis-redis-node-1-1 redis-cli

# Connect to node 1 and view cluster info
docker exec -it redis-redis-node-1-1 redis-cli cluster info

# View cluster nodes
docker exec -it redis-redis-node-1-1 redis-cli cluster nodes
```

### Test the Cluster

```bash
# Set key-value
docker exec -it redis-redis-node-1-1 redis-cli -c set mykey "Hello Redis Cluster"

# Get key-value
docker exec -it redis-redis-node-1-1 redis-cli -c get mykey

# Check which slot the key is in
docker exec -it redis-redis-node-1-1 redis-cli cluster keyslot mykey
```

### Monitoring

```bash
# View all service status
docker compose ps

# View service logs
docker compose logs -f

# View specific service logs
docker compose logs -f redis-node-1

# View resource usage
docker stats
```

## Data Persistence

Each Redis node has its own data volume:

- `redis_node_1_data`
- `redis_node_2_data`
- `redis_node_3_data`
- `redis_node_4_data`
- `redis_node_5_data`
- `redis_node_6_data`

Data is automatically persisted to these volumes and will not be lost even if containers restart.

## Failover

Redis cluster supports automatic failover:

- When a master node fails, the corresponding replica node will be automatically promoted to master
- Cluster node timeout is set to 5000ms
- Health checks are performed every 5 seconds

## Important Notes

1. On first startup, the `cluster-init` service will automatically create the cluster
2. If the cluster already exists, `cluster-init` will detect it and skip the creation step
3. The cluster will only be created after all nodes are healthy
4. Redis Commander will start after the cluster is successfully created

## Troubleshooting

### Cluster Creation Failed

```bash
# Check initialization logs
docker compose logs cluster-init

# Check if all nodes are healthy
docker compose ps

# Reinitialize the cluster
docker compose restart cluster-init
```

### Cannot Connect to Redis Commander

```bash
# Check Redis Commander status
docker compose logs redis-commander

# Check port mapping
docker compose ps redis-commander
```

### Node Cannot Start

```bash
# Check node logs
docker compose logs redis-node-1

# Check if resource limits are too low
# Edit .env file to increase REDIS_MEMORY_LIMIT and REDIS_CPU_LIMIT
```

## License

MIT License

## Contributing

Feel free to submit issues and enhancement requests!
