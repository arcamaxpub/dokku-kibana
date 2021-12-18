# dokku-kibana

![Kibana 4](kibana.png)

Run Kibana on dokku.

## Deploy

Install ElasticSearch and create the app and its db:

```
dokku apps:create kibana
dokku plugin:install https://github.com/dokku/dokku-elasticsearch.git elasticsearch
dokku elasticsearch:create kibana
dokku elasticsearch:link kibana kibana
```

Now push your app:

```
git clone git@github.com:Aluxian/dokku-kibana.git
cd dokku-kibana
git remote add dokku dokku@analytics.arcamax.net:kibana
git push dokku master
```
