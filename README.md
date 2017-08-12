# Foodsaving Tool docker setup

This lets you run everything you need in a docker-compose setup.

## Getting started

```
git clone https://github.com/yunity/foodsaving-docker.git
cd foodsaving-docker
git clone https://github.com/yunity/foodsaving-frontend.git
git clone https://github.com/yunity/foodsaving-backend.git
docker-compose up -d
```

It's up to you to manage your frontend and backend repos, we won't update them. You can switch branches as you desire. You might need to restart frontend or backend after doing so.

## Useful commands

Seed the database:
```
docker-compose exec backend ./manage.py create_sample_data
```

Watch for changes, and run tests in parallel:
```
docker-compose exec backend sh -c "find foodsaving -name '*.py' | entr ./manage.py test --parallel 4 --failfast --keepdb"
```

Restart backend:
```
docker-compose restart backend
```

## Services

Once everything is up and running you can access:

| Name | URL | Description |
|---|---|---|
| main site | [localhost:3000](http://localhost:3000) | |
| swagger backend | [localhost:8000/docs](http://localhost:8000/docs) | |
| maildev | [localhost:1080](http://localhost:1080) | catchs emails sent by app |
| pgadmin4 | [localhost:5050](http://localhost:5050) | see/view/manage postgres database |
| grafana | [localhost:4000](http://localhost:4000) | view stats in influxdb, see "stats" section below |

## stats

Foodsaving Tool reports stats to influxdb using
[django-influxdb-metrics](https://github.com/bitlabstudio/django-influxdb-metrics).

We can use grafana to visualize them.

### influxdb

Create a database:
```
curl -i -XPOST http://localhost:8086/query --data-urlencode "q=CREATE DATABASE fstool"
```

### grafana

Configure datasource:
```
curl -XDELETE http://admin:admin@localhost:4000/api/datasources/name/influxdb
curl -XPOST -H 'Content-Type: application/json' http://admin:admin@localhost:4000/api/datasources -d@stats/datasource.json
```

Configure dashboard:
```
curl -XDELETE http://admin:admin@localhost:4000/api/dashboards/db/requests
curl -XPOST -H 'Content-Type: application/json' http://admin:admin@localhost:4000/api/dashboards/db -d@stats/dashboard.json
```