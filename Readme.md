# dokku-kibana

![Kibana 4](kibana.png)

Run Kibana 7.14.2 on dokku. The Kibana version is determined by the buildpack defined in the .buildpacks file.

## Deploy

Step 1: Install ElasticSearch and create the app and its db:

```
dokku plugin:install https://github.com/dokku/dokku-elasticsearch.git elasticsearch
dokku elasticsearch:create elasticsearch
```

Step 2: Enter the elasticsearch instance and configure auth:

```
dokku elasticsearch:enter elasticsearch
sed -i 5,6d config/elasticsearch.yml # delete cluster master node since this is single node
echo "discovery.type: single-node" >> config/elasticsearch.yml
echo "xpack.security.enabled: true" >> config/elasticsearch.yml
echo "xpack.security.transport.ssl.enabled: true" >> config/elasticsearch.yml
echo "xpack.security.authc.api_key.enabled: true" >> config/elasticsearch.yml
exit

dokku elasticsearch:restart elasticsearch

dokku elasticsearch:enter elasticsearch
# save the output from the following command
# this may fail to connect at first - keep trying
bin/elasticsearch-setup-passwords auto -u "http://localhost:9200"
exit
```

Step 3: Create the kibana app:

```
dokku apps:create kibana
dokku apps:set kibana KIBANA_SYSTEM_PASS=<insert password from above>
dokku elasticsearch:link elasticsearch kibana
```

Step 4: Now push the kibana app:

```
git clone git@github.com:arcamaxpub/dokku-kibana.git dokku-kibana
# you'll need the secret dokku_rsa identity file
git remote add dokku dokku@elasticsearch.arcamax.net:kibana
GIT_SSH_COMMAND="ssh -i ~/.ssh/dokku_rsa" git push dokku master
```

Step 5: Optionally, configure users:

1. Log in with the superuser `elastic` and the saved password from Step 2.
2. In the hamburger menu, go to `Management -> Stack Management -> Security -> Roles` & `Management -> Stack Management -> Security -> Users`.
3. Create any roles & users of the system.
