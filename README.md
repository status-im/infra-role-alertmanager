# Description

This role configures [AlertManager](https://prometheus.io/docs/alerting/alertmanager/) to notify people of threshold breaches in rules configured in Prometheus __master__ instance.

# Configuration

The bare minimum should be:
```yml
alertmanager_domain: 'alerts.example.org'
alertmanager_admin_email: 'admin@example.org'

# SMTP fallback
alertmanager_admin_email: 'admin@example.org'
alertmanager_smtp_host: smtp.mail.example.org'
alertmanager_smtp_from: 'alerts@example.org'
alertmanager_smtp_user: 'secret-smtp-user'
alertmanager_smtp_pass: 'secret-smtp-pass'

# VictorOps API
alertmanager_victorops_api_key: 'secret-victorops-api-key'
alertmanager_victorops_routing_key:  'alert-manager'
alertmanager_victorops_service_url: 'https://alert.victorops.com/integrations/generic/123123123/alert/'
```
Take note you will have to create an `alert-manager` routing rule in VictorOps.

There is also optional OAuth Proxy configuration:
```yaml
alertmanager_oauth_id: '123qwe123qwe'
alertmanager_oauth_secret: '123qwe123qwe123qwe123qwe'
alertmanager_oauth_cookie_secret: '123qwe'
alertmanager_oauth_gh_org: 'my-gh-org'
```

# Management

You can manage existing alerts by using the `amtool` on any of the hosts running this:
```
 > amtool alert
Alertname        Starts At                Summary
Test_Alert       2018-07-06 18:30:18 UTC  This is a testing alert!
```
```
 > amtool silence
ID                                    Matchers                Starts At                Ends At                  Updated At               Created By  Comment  
9635b573-5177-4601-a3b0-ac6a25d0a4ef  alertname=InstanceDown  2018-07-06 12:37:04 UTC  2018-07-06 14:36:05 UTC  2018-07-06 12:37:04 UTC  jakubgs     test
```

# Details

AlertManager runs in a cluster to achieve high availability. The peer connect via [Tinc VPN](https://github.com/status-im/infra-role-bootstrap/tree/master/tasks/tinc).
The service listens on `:9093` and the Prometheus instance connects to that port via the VPN to inform it of threshold breaches.

The main configuration resides in [`templates/alertmanager.yml.j2`](templates/alertmanager.yml.j2).
It configures all the receivers of alerts generated by Prometheus __master__ instance.

The are three main sections:

* `global` - Configure general auth related options for SMTP and Slack receivers.
* `receivers` - Defines destinations of alets which can be used in the `route` section.
* `route` - Defines rules based on which alerts are directed to defined receivers.

For more details see: https://prometheus.io/docs/alerting/configuration/
