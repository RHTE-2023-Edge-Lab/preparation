# Grafana

## Deploy Grafana locally

**/etc/yum.repos.d/grafana.repo**

```ini
[grafana]
name=grafana
baseurl=https://rpm.grafana.com
repo_gpgcheck=1
enabled=1
gpgcheck=1
gpgkey=https://rpm.grafana.com/gpg.key
sslverify=1
sslcacert=/etc/pki/tls/certs/ca-bundle.crt
```

```sh
sudo dnf install grafana
sudo systemctl daemon-reload
sudo systemctl start grafana-server
sudo systemctl status grafana-server
```

## Configure PostgreSQL

```sh
sudo systemctl start postgresql
```

```
$ sudo -u postgres psql template1

psql (14.3)
Saisissez « help » pour l'aide.

template1=# CREATE USER shipments WITH ENCRYPTED PASSWORD 'shipments';
CREATE ROLE

template1=# CREATE DATABASE shipments WITH OWNER 'shipments';
CREATE DATABASE
```

```sh
psql -U shipments -h localhost shipments
```

```sql
CREATE TABLE loc (
    ts TIMESTAMP PRIMARY KEY DEFAULT now(),
    lat DOUBLE PRECISION NOT NULL,
    lon DOUBLE PRECISION NOT NULL
);

INSERT INTO loc (lat, lon) VALUES (48.8147,2.5024);
```

```
$ psql -U shipments -h localhost shipments -c "SELECT * FROM loc;"

Mot de passe pour l'utilisateur shipments : 
             ts             |   lat   |  lon   
----------------------------+---------+--------
 2022-12-07 17:15:12.576841 | 48.8147 | 2.5024
```

Create a Datasource in Grafana.

* Host: localhost:5432
* Database: shipments
* User: shipments
* Password: shipments
* TLS mode: disable

Add a Geomap panel and use the following query:

```sql
SELECT
  ts,
  lat,
  lon
FROM
  loc
LIMIT
  50
```

## Configure MQTT (DEAD END)

Connect to localhost on port 3000 with admin / admin.

```sh
git clone https://github.com/grafana/mqtt-datasource.git
cd mqtt-datasource
sudo dnf install yarn golang-github-magefile-mage
yarn install
yarn build
sudo cp -r dist /var/lib/grafana/plugins/grafana-mqtt-datasource
```

In **/etc/grafana/grafana.ini**:

```ini
app_mode = development

[paths]
plugins = /var/lib/grafana/plugins
```

```sh
sudo systemctl restart grafana-server
```

Create an MQTT datasource

* URI: tcp://test.mosquitto.org:1883

=> Lots of errors in the logs

```
level=error msg="MQTT Connection lost" error=EOF
```

## Deploy Grafana on OpenShift (DEAD END)

Create a project named **grafana**.

Install the Grafana community operator in this project.

```yaml
apiVersion: integreatly.org/v1alpha1
kind: Grafana
metadata:
  name: grafana
  namespace: grafana
spec:
  config:
    security:
      admin_password: secret
      admin_user: admin
    log:
      level: info
      mode: console
    auth.anonymous:
      enabled: true
    auth:
      disable_signout_menu: true
  dataStorage:
    class: managed-nfs-storage
    accessModes:
    - ReadWriteMany
    size: 1Gi
  dashboardLabelSelector:
  - matchExpressions:
    - key: app
      operator: In
      values:
      - grafana
```

```sh
oc create route edge grafana --service=grafana-service -n grafana
```
