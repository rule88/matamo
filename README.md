# docker-compose file for Matamo

A simple compose file for hosting [Matamo](https://matomo.org/). This compose takes into account backup-able volumes (avoid storing tmp/cache data in persistent volume).

## backup

The `db-backup` volume is used to store mysql dumps. With the following lines you can easily create a gezipped database backup stored in /backup for each container that is labelled `mysql.backup=True`:
```bash
for container in $(docker ps --filter "label=mysql.backup=true" --format '{{.Names}}'); do
	echo "dumping database of container $container"
	docker exec $container bash -c 'mysqldump -u root -p$MYSQL_ROOT_PASSWORD $MYSQL_DATABASE | gzip > /backup/$MYSQL_DATABASE-$(date +%u).sql.gz'; 
done
```

The next step is to create the actual backup with your backup tool of choise, volume label `backup=True` will be of help in this step as the following will return all host paths to the volumes that should be backed up:
```bash
docker volume ls --filter "label=backup=True" --format '{{.Mountpoint}}'
```

## Traefik

Traefik is used for routing and managing certificates with let's encrypt. Configuring Traefik is out of scope of this project and can easily left out by removing the labels on the `web` service.

