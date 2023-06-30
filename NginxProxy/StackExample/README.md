# Docker stack example

## Deploying the stacks

1. Deploy the proxy stack
```
docker stack deploy -c docker-compose.proxy.yml grimoire-proxy
```

_If you've never ran stack commands before, it'll ask you to init stack first._

2. Deploy the database stack
```
docker stack deploy -c docker-compose.db.yml grimoire-db
```

3. Deploy the grimoire stacks
```
docker stack deploy -c docker-compose.grimoire-gt5p.yml grimoire-gt5p
docker stack deploy -c docker-compose.grimoire-gt5.yml grimoire-gt5
docker stack deploy -c docker-compose.grimoire-gt6.yml grimoire-gt6
```

## Useful commands

### Docker
#### List deployed services
```
docker service ls
```

#### Scale service
```
docker service scale <serviceName>=<replicaAmount>
```

#### List deployed containers
```
docker ps
```

#### Exec in container
```
docker exec -it <containerId> sh
```
or
```
docker exec -it <containerId> bash
```

### Postgres

#### Login
```
psql -U grimoire
```
