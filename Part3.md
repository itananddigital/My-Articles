### Install ERPNext or any Custom Frappe Apps

#### Create first bench

Create first bench called `erpnext-one` with `test1.8848digital.com` and `test2.8848digital.com`

Create a file called `erpnext-one.env` in `~/gitops`

```shell
cp example.env ~/gitops/erpnext-one.env
sed -i 's/DB_PASSWORD=123/DB_PASSWORD=YOUR_PASSWORD/g' ~/gitops/erpnext-one.env
sed -i 's/DB_HOST=/DB_HOST=mariadb-database/g' ~/gitops/erpnext-one.env
sed -i 's/DB_PORT=/DB_PORT=3306/g' ~/gitops/erpnext-one.env
sed -i 's/SITES=`erp.example.com`/SITES=\`test1.8848digital.com\`,\`test2.8848digital.com\`/g' ~/gitops/erpnext-one.env
echo 'ROUTER=erpnext-one' >> ~/gitops/erpnext-one.env
echo "BENCH_NETWORK=erpnext-one" >> ~/gitops/erpnext-one.env
```

Note:

- Change the password from `YOUR_PASSWORD` to the one set for MariaDB compose in the previous step.

env file is generated at location `~/gitops/erpnext-one.env`.

Create a yaml file called `erpnext-one.yaml` in `~/gitops` directory:

```shell
docker compose --project-name erpnext-one \
  --env-file ~/gitops/erpnext-one.env \
  -f compose.yaml \
  -f overrides/compose.redis.yaml \
  -f overrides/compose.multi-bench.yaml \
  -f overrides/compose.multi-bench-ssl.yaml config > ~/gitops/erpnext-one.yaml
```

For LAN setup do not override `compose.multi-bench-ssl.yaml`.

Use the above command after any changes are made to `erpnext-one.env` file to regenerate `~/gitops/erpnext-one.yaml`. e.g. after changing version to migrate the bench.

Deploy `erpnext-one` containers:

```shell
docker compose --project-name erpnext-one -f ~/gitops/erpnext-one.yaml up -d
```

Create sites `one.example.com` and `two.example.com`:

```shell
# test1.8848digital.com
docker compose --project-name erpnext-one exec backend \
  bench new-site test1.8848digital.com --no-mariadb-socket --mariadb-root-password changeit --install-app erpnext --admin-password changeit
```

You can stop here and have a single bench single site setup complete. Continue to add one more site to the current bench.

```shell
# test2.8848digital.com
docker compose --project-name erpnext-one exec backend \
  bench new-site test2.8848digital.com --no-mariadb-socket --mariadb-root-password changeit --install-app erpnext --admin-password changeit
```
