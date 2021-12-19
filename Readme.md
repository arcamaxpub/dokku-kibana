# dokku-kibana

![Kibana 4](kibana.png)

Run Kibana 7.14.2 on dokku. The Kibana version is determined by the buildpack defined in the .buildpacks file.

## Deploy

Step 1: Install ElasticSearch and create the app and its db:

```
dokku plugin:install https://github.com/dokku/dokku-elasticsearch.git elasticsearch
dokku elasticsearch:create analytics
```

Step 2: Enter the elasticsearch instance and configure auth:

```
dokku elasticsearch:enter analytics
sed -i 5,6d config/elasticsearch.yml # delete cluster master node since this is single node
echo "discovery.type: single-node" >> config/elasticsearch.yml
echo "xpack.security.enabled: true" >> config/elasticsearch.yml
echo "xpack.security.transport.ssl.enabled: true" >> config/elasticsearch.yml
exit

dokku elasticsearch:restart analytics

dokku elasticsearch:enter analytics
# save the output from the following command
# this may fail to connect at first - keep trying
bin/elasticsearch-setup-passwords auto -u "http://localhost:9200"
exit
```

Step 3: Create the kibana app:

```
dokku apps:create kibana
dokku apps:set kibana KIBANA_SYSTEM_PASS=<insert password from above>
dokku elasticsearch:link analytics kibana
```

Step 4: Now push the kibana app:

```
git clone git@github.com:arcamaxpub/dokku-kibana.git dokku-kibana
# you'll need the secret dokku_rsa identity file
git remote add dokku dokku@analytics.arcamax.net:kibana
GIT_SSH_COMMAND="ssh -i ~/.ssh/dokku_rsa" git push dokku master
```

Step 5: Configure users:

1. Log in with the superuser `elastic` and the saved password from Step 2.
2. In the hamburger menu, go to `Management -> Stack Management -> Security -> Users`.
3. Create any users of the system with the role: `editor`.
4. Log out of the superuser account.
