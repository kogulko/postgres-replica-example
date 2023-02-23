### Steps

- Set up primary postgres instance
  ```shell
  docker-compose up -d  postgres_primary
  ```
- Connect to primary postgres instance and create a replication user and replica slot
  ```sql
  CREATE USER replicator WITH REPLICATION ENCRYPTED PASSWORD 'my_replicator_password';
  SELECT * FROM pg_create_physical_replication_slot('replication_slot_secondary1');
  ```
- Create a backup on primary instance
  ```shell
  pg_basebackup -D /var/lib/postgresql/data-secondary -S replication_slot_secondary1 -X stream -P -U replicator -Fp -R
  ```
- Create copy postgresql.auto.conf and pg_hba.conf to wathed directory
  ```shell
  cp /etc/postgresql/init-script/secondary-config/* /var/lib/postgresql/data-secondary
  cp /etc/postgresql/init-script/config/pg_hba.conf /var/lib/postgresql/data
  ```

- Restart primary postgres instance
  ```shell
  docker-compose restart postgres_primary
  ```

- Set up secondary postgres instance
  ```shell
  docker-compose up -d  postgres_secondary
  ```

- Create test table and make sure that replication works
  ```sql
  CREATE TABLE users (firstname text, lastname text, id serial primary key);
   INSERT INTO users (firstname, lastname) VALUES ('Joe', 'Cool') RETURNING id;
  ```
