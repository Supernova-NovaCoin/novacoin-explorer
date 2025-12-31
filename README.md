# NovaCoin Explorer

Ethereum-compatible blockchain explorer powered by Blockscout.

## Quick Start

### Prerequisites

- Docker 20.10+
- Docker Compose 2.0+
- Running NovaCoin node with RPC enabled

### Testnet Deployment

```bash
# Start explorer with testnet configuration
cd explorer
NETWORK=testnet docker-compose up -d

# View logs
docker-compose logs -f backend

# Stop explorer
docker-compose down
```

### Mainnet Deployment

```bash
# Generate secure secrets first
export DB_PASSWORD=$(openssl rand -hex 32)
export SECRET_KEY_BASE=$(openssl rand -hex 64)

# Start with mainnet configuration
NETWORK=mainnet docker-compose up -d
```

## Services

| Service | Port | Description |
|---------|------|-------------|
| Backend (API) | 4000 | Main explorer API |
| Frontend | 3000 | Web interface |
| Smart Contract Verifier | 8050 | Contract verification |
| Visualizer | 8151 | Contract visualization |
| Stats | 8153 | Blockchain statistics |

## Configuration

### Environment Variables

| Variable | Default | Description |
|----------|---------|-------------|
| `NETWORK` | testnet | Network configuration (testnet/mainnet) |
| `RPC_URL` | http://host.docker.internal:8545 | Node RPC endpoint |
| `WS_URL` | ws://host.docker.internal:8546 | Node WebSocket endpoint |
| `CHAIN_ID` | 1 | Chain ID |
| `EXPLORER_PORT` | 4000 | Backend API port |
| `FRONTEND_PORT` | 3000 | Frontend web port |
| `DB_PASSWORD` | blockscout_password | PostgreSQL password |

### Custom Configuration

Edit files in `config/` directory:

- `common.env` - Shared settings for all environments
- `testnet.env` - Testnet-specific settings
- `mainnet.env` - Mainnet-specific settings

## API Usage

### REST API

```bash
# Get latest blocks
curl http://localhost:4000/api/v2/blocks

# Get transaction
curl http://localhost:4000/api/v2/transactions/{hash}

# Get address info
curl http://localhost:4000/api/v2/addresses/{address}

# Get token info
curl http://localhost:4000/api/v2/tokens/{address}
```

### GraphQL API

Available at `http://localhost:4000/graphiql`

## Maintenance

### Database Backup

```bash
# Backup
docker-compose exec db pg_dump -U blockscout blockscout > backup.sql

# Restore
docker-compose exec -T db psql -U blockscout blockscout < backup.sql
```

### Clear Cache

```bash
docker-compose exec redis redis-cli FLUSHALL
```

### Reindex Blocks

```bash
# Stop indexer
docker-compose stop backend

# Clear indexed data (keeps settings)
docker-compose exec db psql -U blockscout -c "TRUNCATE blocks CASCADE;"

# Restart
docker-compose start backend
```

## Troubleshooting

### Explorer Not Syncing

1. Check RPC connectivity:
   ```bash
   curl -X POST -H "Content-Type: application/json" \
     --data '{"jsonrpc":"2.0","method":"eth_blockNumber","params":[],"id":1}' \
     http://localhost:8545
   ```

2. Check explorer logs:
   ```bash
   docker-compose logs backend | grep -i error
   ```

3. Verify database connection:
   ```bash
   docker-compose exec db psql -U blockscout -c "SELECT 1;"
   ```

### High Memory Usage

Adjust memory limits in docker-compose.yml:
```yaml
backend:
  deploy:
    resources:
      limits:
        memory: 4G
```

## Related Repositories

- [novacoin](https://github.com/novacoin/novacoin) - Core Blockchain
- [novacoin-sdk-ts](https://github.com/novacoin/novacoin-sdk-ts) - TypeScript SDK
- [novacoin-docs](https://github.com/novacoin/novacoin-docs) - Documentation

## License

MIT
