# Recommended node settings 

Some dockers require specific host settings, especially all dockers having something todo with ElasticSearch. 

in `/etc/sysctl.d` folder, make sure to create a `10-es.conf` file with the following line:
`vm.max_map_count=262144` 


