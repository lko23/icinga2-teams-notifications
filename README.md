<!--
  Title: Icinga Teams Notifications
  Description: Icinga 2 notification integration with Teams
  Author: nisabek richard.hauswald
  -->

# icinga2-teams-notifications
Icinga2 notification integration with Teams

## Overview

Native, easy to use Icinga2 `NotificationCommand` to send Host and Service notifications to pre-configured Teams channel - with only 1 external dependency: `curl`

## What will I get?
* Awesome Teams notifications:

<p align="center">
  <img src="https://github.com/lko23/icinga2-teams-notifications/raw/master/Notification-Example.png" width="600">
</p>

* Notifications inside Teams about your Host and Service state changes
* In case of failure get notified with the nicely-formatted output of the failing check
* Easy integration with Icinga2
* Only native Icinga2 features are used, no bash, perl, etc - keeps your server/virtual machine/docker instances small
* Debian ready-to-use package to reduce maintenance and automated installation effort
* Uses Lambdas!

## Why another approach?

We found the following 2 existing Icinga2 to Teams integrations. 

* https://github.com/spjmurray/Teams-icinga2

  This plugin provides a polling interface towards Icinga2, giving the possibility to query the Icinga2 API and get information. 
  
  Since we cannot open our firewalls to enable access for Teams servers to our Icinga2 instances, we need to
  have Icinga2 sending push notifications to Teams to report our Host and Service state changes.
* https://github.com/jjethwa/icinga2-Teams-notification
  
  The plugin provides the possibility to send NotificationCommand to Teams, however we found the following 
  downsides to be show stoppers for us:
   * Does not use Lambdas!
   * The Integration is time consuming and cumbersome
   * The author requires you to modify his source files in order to configure the Teams webhook and channel. So we'd 
   have to configure everything again when we have to install an update of that integration.
   * No Debian package available, which leads to increased installation and maintenance effort.
   * Numerous bugs since the host output is not properly encoded:
     * as shell argument before it's passed to the shell script
     * as JSON before it's send to Teamss REST API

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
foo@bar:~# add-apt-repository "deb https://raw.githubusercontent.com/nisabek/icinga2-Teams-notifications/master/reprepro general main"
foo@bar:~# apt-get update
```

You are now ready to install the plugin with 

```console
foo@bar:~# apt-get install icinga2-Teams-notifications
```

This will create the plugin files in the correct `icinga2` conf directory. 

### Installation using git

1. Clone the repository and copy the relevant folder into your Icinga2 `/etc/icinga2/conf.d` directory
 
```console
foo@bar:~# git clone git@github.com:nisabek/icinga2-Teams-notifications.git /opt/icinga2-Teams-notifications
foo@bar:~# cp -r /opt/icinga2-Teams-notifications/src/Teams-notifications /etc/icinga2/conf.d/
```

2. Use the `Teams-notifications-user-configuration.conf.template` file as reference to configure your Teams Webhook URL and Icinga2 Base URL to create your own
 `Teams-notifications-user-configuration.conf`
 
```console
foo@bar:~# cp /etc/icinga2/conf.d/Teams-notifications/Teams-notifications-user-configuration.conf.template /etc/icinga2/conf.d/Teams-notifications/Teams-notifications-user-configuration.conf
```
 
3. Fix permissions
 
```console
foo@bar:~# chown -R root:nagios /etc/icinga2/conf.d/Teams-notifications
foo@bar:~# chmod 0750 /etc/icinga2/conf.d/Teams-notifications
foo@bar:~# chmod 0640 /etc/icinga2/conf.d/Teams-notifications/*
```

### Configuration 
 
#### Icinga2 features

In order for the Teams-notifications to work you need at least the `checker`,  `command` and `notification` icinga2 features enabled.

In order to see the list of currently enabled features execute the following command

```console
foo@bar:~# icinga2 feature list
```

In order to enable a feature use 

```console
foo@bar:~# icinga2 feature enable FEATURE_NAME
```

#### Notification configuration

1. Configure Teams Webhook and Icinga2 web URLs in `/etc/icinga2/conf.d/Teams-notifications/Teams-notifications-user-configuration.conf`
```php
template Notification "Teams-notifications-user-configuration" {
    import "Teams-notifications-default-configuration"

    vars.Teams_notifications_webhook_url = "<YOUR Teams WEBHOOK URL>, e.g. https://hooks.Teams.com/services/TOKEN1/TOKEN2"
    vars.Teams_notifications_icinga2_base_url = "<YOUR ICINGA2 BASE URL>, e.g. http://icinga-web.yourcompany.com/icingaweb2"
}
...
```

2. In order to enable the Teams-notifications **for Services** add `vars.Teams_notifications = "enabled"` to your Service template, e.g. in `/etc/icinga2/conf.d/templates.conf`

```php
 template Service "generic-service" {
   max_check_attempts = 5
   check_interval = 1m
   retry_interval = 30s
 
   vars.Teams_notifications = "enabled"
 }
 ```

In order to enable the Teams-notifications **for Hosts** add `vars.Teams_notifications = "enabled"` to your Host template, e.g. in `/etc/icinga2/conf.d/templates.conf`

```php
 template Host "generic-host" {
   max_check_attempts = 5
   check_interval = 1m
   retry_interval = 30s
 
   vars.Teams_notifications = "enabled"
 }
 ```

Make sure to restart icinga after the changes

```console
foo@bar:~# systemctl restart icinga2
```

2. Further customizations [_optional_]

You can customize the following parameters of Teams-notifications :
  * Teams_notifications_channel [Default: `#monitoring_alerts`]
  * Teams_notifications_botname [Default: `icinga2`]
  * Teams_notifications_plugin_output_max_length [Default: `3500`]
  * Teams_notifications_icon_dictionary [Default:
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

In order to do so, place the desired parameter into `Teams-notifications-user-configuration.conf` file.

Note 
> Objects as well as templates themselves can import an arbitrary number of other templates. Attributes inherited from a template can be overridden in the object if necessary.

The `Teams-notifications-user-configuration` section applies to both Host and Service, whereas the 
`Teams-notifications-user-configuration-hosts` and `Teams-notifications-user-configuration-services` sections apply to Host and Service respectively


_Example channel name configuration for Service notifications_

```php
template Notification "Teams-notifications-user-configuration" {
    import "Teams-notifications-default-configuration"

    vars.Teams_notifications_webhook_url = "<YOUR Teams WEBHOOK URL>, e.g. https://hooks.Teams.com/services/TOKEN1/TOKEN2"
    vars.Teams_notifications_icinga2_base_url = "<YOUR ICINGA2 BASE URL>, e.g. http://icinga-web.yourcompany.com/icingaweb2"
}

template Notification "Teams-notifications-user-configuration-hosts" {
    import "Teams-notifications-default-configuration-hosts"

    interval = 1m
}

template Notification "Teams-notifications-user-configuration-services" {
    import "Teams-notifications-default-configuration-services"

    interval = 3m
    
    vars.Teams_notifications_channel = "#monitoring_alerts_for_service"
}
```

You can choose to override the whole icon dictionary, or override specific types only:

_Example override the whole icon dictionary_

```php
template Notification "Teams-notifications-user-configuration" {
    import "Teams-notifications-default-configuration"

    vars.Teams_notifications_webhook_url = "https://hooks.Teams.com/services/T2T1TT1LL/B4GESBE48/ao4UYahfe1FkRPhlRKWzf6uu"
    vars.Teams_notifications_icinga2_base_url = "http://localhost:80/icingaweb2"
    vars.Teams_notifications_channel = "#icinga2-private-test"
    vars.Teams_notifications_icon_dictionary = {
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
template Notification "Teams-notifications-user-configuration" {
    import "Teams-notifications-default-configuration"

    vars.Teams_notifications_webhook_url = "https://hooks.Teams.com/services/T2T1TT1LL/B4GESBE48/ao4UYahfe1FkRPhlRKWzf6uu"
    vars.Teams_notifications_icinga2_base_url = "http://localhost:80/icingaweb2"
    vars.Teams_notifications_channel = "#icinga2-private-test"

    vars.Teams_notifications_icon_dictionary.CUSTOM = "cherries"
    ...
}
```

If you, for some reason, want to disable the Teams-notifications from icinga2 change the following parameter inside the 
corresponding Host or Service configuration object/template:

`vars.Teams_notifications == "disabled"`

Besides configuring the Teams-notifications parameters you can also configure other Icinga2 specific configuration 
parameters of the Host and Service, e.g.:
* types
* user_groups
* interval
* period

## How it works
Teams-notifications uses the icinga2 native [NotificationCommand] (https://docs.icinga.com/icinga2/latest/doc/module/icinga2/chapter/object-types#objecttype-notificationcommand) 
to collect the required data and send a message to configured Teams channel using `curl`

The implementation can be found in `Teams-notifications-command.conf` and it uses Lambdas!

## Testing

Since the official docker image of icinga2 seems not to be maintained, we've been using [jordan's icinga2 image](https://hub.docker.com/r/jordan/icinga2/)
to test the notifications manually.

Usual procedure for us to test the plugin is to 

* configure the `src/Teams-notifications/Teams-notifications-configuration.conf` file according to documentation
* configure a test `src/templates.conf` which contains the Teams-notifications enabled for host and/or service
* run the `jordan/icinga2` with an empty volume at first
* copy the configurations to relevant directories
* restart the container

```console
foo@bar:~# docker run -p 8081:80 --name Teams-enabled-icinga2 -v $PWD/icinga2-docker-volume:/etc/icinga2 -idt jordan/icinga2:latest
foo@bar:~# docker cp src/templates.conf Teams-enabled-icinga2:/etc/icinga2/conf.d/
foo@bar:~# docker cp src/Teams-notifications Teams-enabled-icinga2:/etc/icinga2/conf.d/
foo@bar:~# docker restart Teams-enabled-icinga2
```

after that navigate to `http://localhost:8081/icingaweb2` and try out some notifications. 

We understand that this is far from automated testing, and we will be happy to any contributions that would improve the procedure.

## Troubleshooting
The Teams-notifications command provides detailed debug logs. In order to see them, make sure the `debuglog` feature of icinga2 is enabled.

```console
foo@bar:~# icinga2 feature enable debuglog
```

After that you should see the logs in `/var/log/icinga2/debug.log` file. All the Teams-notifications specific logs are pre-pended with "debug/Teams-notifications"

Use the following grep for troubleshooting: 

```console
foo@bar:~# grep "warning/PluginNotificationTask\|Teams-notifications" /var/log/icinga2/debug.log
foo@bar:~# tail -f /var/log/icinga2/debug.log | grep "warning/PluginNotificationTask\|Teams-notifications"
```

## Useful links
- [Setup Teams Webhook](https://api.Teams.com/incoming-webhooks)
- [Enable Icinga2 Debug logging](https://docs.icinga.com/icinga2/latest/doc/module/icinga2/chapter/troubleshooting)
- [NotificationCommand of Icinga2](https://docs.icinga.com/icinga2/latest/doc/module/icinga2/toc#!/icinga2/latest/doc/module/icinga2/chapter/object-types#objecttype-notificationcommand)
- [Overriding template definitions of Icinga2](https://docs.icinga.com/icinga2/latest/doc/module/icinga2/toc#!/icinga2/latest/doc/module/icinga2/chapter/monitoring-basics#object-inheritance-using-templates)
- [Dockerized Icinga2](https://hub.docker.com/r/jordan/icinga2/)

## Running with Icinga Director

There has been some discussion [over here](https://github.com/nisabek/icinga2-Teams-notifications/issues/5) on how to run the plugin with Icinga Director. We'd appreciate somebody going over this part of documentation and verifying it. 
 
Main points to make it work:

* Use the git version, not the debian package.
* Create a custom variable for Teams_notifications as a string. 
* Run the kickstart wizard: https://github.com/nisabek/icinga2-Teams-notifications/issues/5#issuecomment-369571754

### Latest how-to on Icinga Director

Thanks a lot to @jwhitbread for putting down these steps in the [issue](https://github.com/nisabek/icinga2-Teams-notifications/issues/32)

* Use the git version, not the debian package.
* Add all three files into your global_templates directory e.g /etc/icinga2/zones.d/global_templates/. (I believe these can also be put into your director/master directory)
* Follow the above tutorial with editing said files and permissions.
* Add the vars.Teams_notifications = "enabled" to the relevant templates or directly to the host/service objects.
* Restart icinga2 - systemctl restart icinga2.
* Navigate over to your icinga web interface > Icinga director on the left > Deployments
* This should notify you that you have a change, click deploy. If it doesn't then on the same page navigate to Render config > Deploy anyways.
* Check the console output of the deployment, green tick = good! Go double check your hosts and services are picking up the new variable and then give it a test!
