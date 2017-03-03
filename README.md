# check_junos_ipmonitoring.py

This Script is intended for Nagios or Icinga to monitor the Status of an ip-monitoring.

## Requirements
* Python3
* junos-eznc

## Command Line Options
* -r,--router <host> - Hostname or IP of Router to query
* -u,--user <username> - NETCONF User
* -p,--password <password> - NETCONF Password
* -t,--test <name> - Name of the policy to check

## Examples

python3 check_junos_ipmonitoring.py -r srx.corp.local -u admin -p password -t failover

### Icinga2 check
```
object CheckCommand "junos-ipmonitoring" {
    import "plugin-check-command"
    object CheckCommand "junos-ipmonitoring" {
    import "plugin-check-command"
    command = [ CustomPluginDir + "/check_junos_ipmonitoring.py" ]
    arguments = {
        "-r" = {
            description = "HOST"
            value = "$address$"
        }
        "-u" = {
            description = "netconf user"
            value = "$junos_netconf_user$"
        }
        "-p" = {
            description = "netconf password"
            value = "$junos_netconf_password$"
        }
        "-t" = {
            description = "policy name"
            value = "$junos_ipmonitoring_policy$"
        }
    }
}

object Service "ip monitoring failover" {
    import "generic-service"
    host_name = "srx.corp.local"
    check_command = "junos-ipmonitoring"
    vars.junos_netconf_user = "admin"
    vars.junos_netconf_password = "password"
    vars.junos_ipmonitoring_policy = "failover"
}

```

### Juniper Configuration Example
* Netconf
```
set system services netconf ssh
```

* ip-monitoring check
```
# define a test
set services rpm probe pingcheck test 8.8.8.8 target address 8.8.8.8
set services rpm probe pingcheck test 8.8.8.8 probe-count 5
set services rpm probe pingcheck test 8.8.8.8 probe-interval 1
set services rpm probe pingcheck test 8.8.8.8 test-interval 60
set services rpm probe pingcheck test 8.8.8.8 thresholds successive-loss 3
set services rpm probe pingcheck test 8.8.8.8 next-hop 10.0.0.1
# define ip-monitoring policy (the policy name "failover" is what is used on the -t flag)
set services ip-monitoring policy failover match rpm-probe pingcheck
set services ip-monitoring policy failover then preferred-route route 0.0.0.0/0 next-hop 192.0.2.1
```

## License
This project is licensed under the CC0 License.
