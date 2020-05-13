# Logrotate role

## Role Variables

`logrotate_scripts`: A list of logrotate scripts and the directives to use for the rotation.

`hourly_logrotate_scripts`: A list of logrotate scripts and directives for hourly rotation.

name - The name of the script that goes into /etc/logrotate.d/ or /etc/logrotate.hourly.d/
paths - Path to point logrotate to for the log rotation
options - List of directives for logrotate, view the logrotate man page for specifics
scripts - Dict of scripts for logrotate (see Example below)

## Standard logrotate configuration
Typical logrotate configuration set up in group_vars or host_vars for specific server/group of servers:

```
logrotate_scripts:
  - name: nginx
    confs:
      - paths:
        - /var/log/nginx/*.log
        options:
          - daily
          - missingok
          - size 10M
          - rotate 7
          - compress
          - delaycompress
          - compresscmd /bin/bzip2
          - compressoptions -4
          - compressext .bz2
          - notifempty
          - dateext
          - dateformat _%Y-%m-%d_%H_%M
          - create 0640 www-data www-pub
          - sharedscripts
          - postrotate if [ -f /var/run/nginx.pid ]; then kill -USR1 `cat /var/run/nginx.pid`; fi
          - endscript
```

## Hourly logrotate
All tasks with tag hourly_logs are dedicated to case when we need to trigger logs rotation every hour.

#### Requirements:
Add this to group_vars or host_vars for specific server/group of servers:

```
hourly_logrotate: true
hourly_logrotate_scripts:
  - name: rsyslog
    confs:
      - paths:
        - /var/log/syslog
        options:
          - daily
          - rotate 70
          - size 10M
          - missingok
          - notifempty
          - delaycompress
          - compress
          - compresscmd /bin/bzip2
          - compressoptions -4
          - compressext .bz2
          - dateext
          - dateformat _%Y-%m-%d_**{% if logrotate_ver is version('3.9.0.', '<') %}%s{% else %}%H_%M{% endif %}**
          - postrotate invoke-rc.d rsyslog rotate > /dev/null
          - endscript
```
