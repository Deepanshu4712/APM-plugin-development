# APM-plugin-development
[SnappyFlow](https://www.snappyflow.io/#/) APM new plugin development guide 

## Steps to add new collectd-agent-plugin in APM

Before starting, a few points to keep in mind.

  a) Maintain naming convention. Name of the agent-code file, configuration file and service name should be exactly same.

  b) No camelCase, SnakeCase or '-' and whitespace allowed. Names should be in lowercase without space. For example : mysql, promethueslinux.

  c) Create a fork of the following git repos:
  
    1. maplelabs/collectd-plugins
    2. maplelabs/configurator-exporter-apm
    3. pramurthy/sf-apm-server
    4. pramurthy/deepInsight-Plugins
  
  d) Code changes mandatorily must be in 'dev' branch in the fork as well.

  e) Sync the fork with the main repo before pushing code changing.

## For Development

1. Create a <<service_name>>.py file under /opt/collectd/plugins/.

2. Create a <<service_name>>.conf file and place it under /opt/collectd/conf/. Checkout sample configuration file under _Samples_ section below. 

3. Include the configuration file in collectd.conf under 

        sudo vi /opt/collectd/etc/collectd.conf 
        
   by adding 

     > Include "/opt/collectd/conf/<<service_name>>.conf"

4. Make sure "\_actual_collectd_plugin_type" key is added in the final result metric dictionary before sending it to elasticsearch.

5. Add the service name to the elasticsearch filter. This is used to filter out the metric-data agent is sending to the elasticsearch based on the plugin service type using the key "\_plugin". Add the service name in the file filter.conf under

        sudo vi /opt/collectd/conf/filter.conf
        
   in the block 
   
     > <Rule "elasticsearch">
     
     >   <Match "regex">
     
     >    Plugin "cpu_static|disk_stat|ram_util|cpu_util|tcp_stats|nic_stats|*<service_name>*"
     
     >   </Match>
     
     >   <Target "write">
     
     >    Plugin "write_http/elasticsearch"
     
     >   </Target>
     
     > </Rule>

5. Add the service name in discovery.py file under 

        sudo vi /opt/configurator-exporter/service_discovery/discovery.py

6. Update the metrics_plugin_mapping.yaml under 

        sudo vi /opt/configurator-exporter/config_handler/mapping/metrics_plugin_mapping.yaml
        
7. Since the controller cannot communicate with the configurator exporter (as per the current design) , make the similar changes in the controller mapping file as well under:
    
        sudo vi /home/deepinsight/deepInsight/deepinsight/metrics_plugins_mapping.yaml
        
8. Restart the collectd and configurator service using:

        sudo service collectd restart
        
        sudo service configurator restart
        
        
9. Done !


After the successful completion of the above mentioned steps without any error, check the logs under 

        vi /tmp/collectd.log
