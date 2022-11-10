<!--
  Title: Icinga teams Notifications
  Description: Icinga 2 notification integration with teams
  Author: nisabek richard.hauswald
  -->

# icinga2-teams-notifications
Icinga2 notification integration with teams

## Overview

Native, easy to use Icinga2 `NotificationCommand` to send Host and Service notifications to pre-configured teams channel - with only 1 external dependency: `curl`

## What will I get?
* Awesome teams notifications:

<p align="center">
  <img src="https://github.com/lko23/icinga2-teams-notifications/raw/master/Notification-Example.png" width="600">
</p>

* Notifications inside teams about your Host and Service state changes
* In case of failure get notified with the nicely-formatted output of the failing check
* Easy integration with Icinga2
* Only native Icinga2 features are used, no bash, perl, etc - keeps your server/virtual machine/docker instances small
* Debian ready-to-use package to reduce maintenance and automated installation effort
* Uses Lambdas!

## Installation 

### Installation using Debian package

> !NOTE: At this moment debian package is behind the master code. If you want to use this plugin with Icinga Director, please make sure to install from Git (see below)
We're working on updating the repo in the meantime. 

We use [reprepro](https://mirrorer.alioth.debian.org/) to distribute our package from github.
You would need to install `apt-transport-https` that supports adding an `https` based repository to the debian repo list.

here are the steps to perform:

```console
foo@bar:~# apt-get install -y apt-transport-https
foo@bar:~# apt-key adv --keyserver keyserver.ubuntu.com --recv-keys 10779AB4
foo@bar:~# add-apt-repository "deb https://raw.githubusercontent.com/nisabek/icinga2-teams-notifications/master/reprepro general main"
foo@bar:~# apt-get update
```

You are now ready to install the plugin with 

```console
foo@bar:~# apt-get install icinga2-teams-notifications
```

This will create the plugin files in the correct `icinga2` conf directory. 

### Installation using git

1. Clone the repository and copy the relevant folder into your Icinga2 `/etc/icinga2/conf.d` directory
 
```console
foo@bar:~# git clone git@github.com:nisabek/icinga2-teams-notifications.git /opt/icinga2-teams-notifications
foo@bar:~# cp -r /opt/icinga2-teams-notifications/src/teams-notifications /etc/icinga2/conf.d/
```

2. Use the `teams-notifications-user-configuration.conf.template` file as reference to configure your teams Webhook URL and Icinga2 Base URL to create your own
 `teams-notifications-user-configuration.conf`
 
```console
foo@bar:~# cp /etc/icinga2/conf.d/teams-notifications/teams-notifications-user-configuration.conf.template /etc/icinga2/conf.d/teams-notifications/teams-notifications-user-configuration.conf
```
 
3. Fix permissions
 
```console
foo@bar:~# chown -R root:nagios /etc/icinga2/conf.d/teams-notifications
foo@bar:~# chmod 0750 /etc/icinga2/conf.d/teams-notifications
foo@bar:~# chmod 0640 /etc/icinga2/conf.d/teams-notifications/*
```

### Configuration 
 
#### Icinga2 features

In order for the teams-notifications to work you need at least the `checker`,  `command` and `notification` icinga2 features enabled.

In order to see the list of currently enabled features execute the following command

```console
foo@bar:~# icinga2 feature list
```

In order to enable a feature use 

```console
foo@bar:~# icinga2 feature enable FEATURE_NAME
```

#### Notification configuration

1. Configure teams Webhook and Icinga2 web URLs in `/etc/icinga2/conf.d/teams-notifications/teams-notifications-user-configuration.conf`
```php
template Notification "teams-notifications-user-configuration" {
    import "teams-notifications-default-configuration"

    vars.teams_notifications_webhook_url = "<YOUR teams WEBHOOK URL>, e.g. https://hooks.teams.com/services/TOKEN1/TOKEN2"
    vars.teams_notifications_icinga2_base_url = "<YOUR ICINGA2 BASE URL>, e.g. http://icinga-web.yourcompany.com/icingaweb2"
}
...
```

2. In order to enable the teams-notifications **for Services** add `vars.teams_notifications = "enabled"` to your Service template, e.g. in `/etc/icinga2/conf.d/templates.conf`

```php
 template Service "generic-service" {
   max_check_attempts = 5
   check_interval = 1m
   retry_interval = 30s
 
   vars.teams_notifications = "enabled"
 }
 ```

In order to enable the teams-notifications **for Hosts** add `vars.teams_notifications = "enabled"` to your Host template, e.g. in `/etc/icinga2/conf.d/templates.conf`

```php
 template Host "generic-host" {
   max_check_attempts = 5
   check_interval = 1m
   retry_interval = 30s
 
   vars.teams_notifications = "team_x"
 }
 ```

Make sure to restart icinga after the changes

```console
foo@bar:~# systemctl restart icinga2
```

2. Further customizations [_optional_]

You can customize the following parameters of teams-notifications :
  * teams_notifications_channel [Default: `#monitoring_alerts`]
  * teams_notifications_botname [Default: `icinga2`]
  * teams_notifications_plugin_output_max_length [Default: `3500`]
  * teams_notifications_icon_dictionary [Default:
   ```php
     {
         "DOWNTIMEREMOVED" = "leftwards_arrow_with_hook",
         "ACKNOWLEDGEMENT" = "ballot_box_with_check",
         "PROBLEM" = "red_circle",
         "RECOVERY" = "large_blue_circle",
         "DOWNTIMESTART" = "arrow_up_small",
         "DOWNTIMEEND" = "arrow_down_small",
         "FLAPPINGSTART" = "small_red_triangle",
         "FLAPPINGEND" = "small_red_triangle_down",
         "CUSTOM" = "speaking_head_in_silhouette"
     }
   ```
  ]

In order to do so, place the desired parameter into `teams-notifications-user-configuration.conf` file.

Note 
> Objects as well as templates themselves can import an arbitrary number of other templates. Attributes inherited from a template can be overridden in the object if necessary.

The `teams-notifications-user-configuration` section applies to both Host and Service, whereas the 
`teams-notifications-user-configuration-hosts` and `teams-notifications-user-configuration-services` sections apply to Host and Service respectively


_Example channel name configuration for Service notifications_

```php
template Notification "teams-notifications-user-configuration" {
    import "teams-notifications-default-configuration"

    vars.teams_notifications_webhook_url = "<YOUR teams WEBHOOK URL>, e.g. https://hooks.teams.com/services/TOKEN1/TOKEN2"
    vars.teams_notifications_icinga2_base_url = "<YOUR ICINGA2 BASE URL>, e.g. http://icinga-web.yourcompany.com/icingaweb2"
}

template Notification "teams-notifications-user-configuration-hosts" {
    import "teams-notifications-default-configuration-hosts"

    interval = 1m
}

template Notification "teams-notifications-user-configuration-services" {
    import "teams-notifications-default-configuration-services"

    interval = 3m
    
    vars.teams_notifications_channel = "#monitoring_alerts_for_service"
}
```

You can choose to override the whole icon dictionary, or override specific types only:

_Example override the whole icon dictionary_

```php
template Notification "teams-notifications-user-configuration" {
    import "teams-notifications-default-configuration"

    vars.teams_notifications_webhook_url = "https://community.webhook.office.com/webhookb2/Token1/IncomingWebhook/Token2"
    vars.teams_notifications_icinga2_base_url = "http://localhost:80/icingaweb2"
    vars.teams_notifications_channel = "#icinga2-private-test"
    vars.teams_notifications_icon_dictionary = {
       "DOWNTIMEREMOVED" = "leftwards_arrow_with_hook",
       "ACKNOWLEDGEMENT" = "ballot_box_with_check",
       "PROBLEM" = "bomb",
       "RECOVERY" = "large_blue_circle",
       "DOWNTIMESTART" = "up",
       "DOWNTIMEEND" = "arrow_double_down",
       "FLAPPINGSTART" = "small_red_triangle",
       "FLAPPINGEND" = "small_red_triangle_down",
       "CUSTOM" = "speaking_head_in_silhouette"
    }    
    ...
```

_Example override specific type_

```php
template Notification "teams-notifications-user-configuration" {
    import "teams-notifications-default-configuration"

    vars.teams_notifications_webhook_url = "https://hooks.teams.com/services/T2T1TT1LL/B4GESBE48/ao4UYahfe1FkRPhlRKWzf6uu"
    vars.teams_notifications_icinga2_base_url = "http://localhost:80/icingaweb2"
    vars.teams_notifications_channel = "#icinga2-private-test"

    vars.teams_notifications_icon_dictionary.CUSTOM = "cherries"
    ...
}
```

If you, for some reason, want to disable the teams-notifications from icinga2 change the following parameter inside the 
corresponding Host or Service configuration object/template:

`vars.teams_notifications == "disabled"`

Besides configuring the teams-notifications parameters you can also configure other Icinga2 specific configuration 
parameters of the Host and Service, e.g.:
* types
* user_groups
* interval
* period

## How it works
teams-notifications uses the icinga2 native [NotificationCommand] (https://docs.icinga.com/icinga2/latest/doc/module/icinga2/chapter/object-types#objecttype-notificationcommand) 
to collect the required data and send a message to configured teams channel using `curl`

The implementation can be found in `teams-notifications-command.conf` and it uses Lambdas!

## Testing

Since the official docker image of icinga2 seems not to be maintained, we've been using [jordan's icinga2 image](https://hub.docker.com/r/jordan/icinga2/)
to test the notifications manually.

Usual procedure for us to test the plugin is to 

* configure the `src/teams-notifications/teams-notifications-configuration.conf` file according to documentation
* configure a test `src/templates.conf` which contains the teams-notifications enabled for host and/or service
* run the `jordan/icinga2` with an empty volume at first
* copy the configurations to relevant directories
* restart the container

```console
foo@bar:~# docker run -p 8081:80 --name teams-enabled-icinga2 -v $PWD/icinga2-docker-volume:/etc/icinga2 -idt jordan/icinga2:latest
foo@bar:~# docker cp src/templates.conf teams-enabled-icinga2:/etc/icinga2/conf.d/
foo@bar:~# docker cp src/teams-notifications teams-enabled-icinga2:/etc/icinga2/conf.d/
foo@bar:~# docker restart teams-enabled-icinga2
```

after that navigate to `http://localhost:8081/icingaweb2` and try out some notifications. 

We understand that this is far from automated testing, and we will be happy to any contributions that would improve the procedure.

## Troubleshooting
The teams-notifications command provides detailed debug logs. In order to see them, make sure the `debuglog` feature of icinga2 is enabled.

```console
foo@bar:~# icinga2 feature enable debuglog
```

After that you should see the logs in `/var/log/icinga2/debug.log` file. All the teams-notifications specific logs are pre-pended with "debug/teams-notifications"

Use the following grep for troubleshooting: 

```console
foo@bar:~# grep "warning/PluginNotificationTask\|teams-notifications" /var/log/icinga2/debug.log
foo@bar:~# tail -f /var/log/icinga2/debug.log | grep "warning/PluginNotificationTask\|teams-notifications"
```

## Useful links
- [Setup teams Webhook](https://api.teams.com/incoming-webhooks)
- [Enable Icinga2 Debug logging](https://docs.icinga.com/icinga2/latest/doc/module/icinga2/chapter/troubleshooting)
- [NotificationCommand of Icinga2](https://docs.icinga.com/icinga2/latest/doc/module/icinga2/toc#!/icinga2/latest/doc/module/icinga2/chapter/object-types#objecttype-notificationcommand)
- [Overriding template definitions of Icinga2](https://docs.icinga.com/icinga2/latest/doc/module/icinga2/toc#!/icinga2/latest/doc/module/icinga2/chapter/monitoring-basics#object-inheritance-using-templates)
- [Dockerized Icinga2](https://hub.docker.com/r/jordan/icinga2/)

## Running with Icinga Director

There has been some discussion [over here](https://github.com/nisabek/icinga2-teams-notifications/issues/5) on how to run the plugin with Icinga Director. We'd appreciate somebody going over this part of documentation and verifying it. 
 
Main points to make it work:

* Use the git version, not the debian package.
* Create a custom variable for teams_notifications as a string. 
* Run the kickstart wizard: https://github.com/nisabek/icinga2-teams-notifications/issues/5#issuecomment-369571754

### Latest how-to on Icinga Director

Thanks a lot to @jwhitbread for putting down these steps in the [issue](https://github.com/nisabek/icinga2-teams-notifications/issues/32)

* Use the git version, not the debian package.
* Add all three files into your global_templates directory e.g /etc/icinga2/zones.d/global_templates/. (I believe these can also be put into your director/master directory)
* Follow the above tutorial with editing said files and permissions.
* Add the vars.teams_notifications = "enabled" to the relevant templates or directly to the host/service objects.
* Restart icinga2 - systemctl restart icinga2.
* Navigate over to your icinga web interface > Icinga director on the left > Deployments
* This should notify you that you have a change, click deploy. If it doesn't then on the same page navigate to Render config > Deploy anyways.
* Check the console output of the deployment, green tick = good! Go double check your hosts and services are picking up the new variable and then give it a test!
