<!-- note marker start -->
**NOTE**: This repo contains only the documentation for the private BoltsOps Pro repo code.
Original file: https://github.com/boltopspro/awslogs/blob/master/README.md
The docs are publish so they are available for interested customers.
For access to the source code, you must be a paying BoltOps Pro subscriber.
If are interested, you can contact us at contact@boltops.com or https://www.boltops.com

<!-- note marker end -->

# Configset awslogs

![CodeBuild](https://codebuild.us-west-2.amazonaws.com/badges?uuid=eyJlbmNyeXB0ZWREYXRhIjoicStFa2owd0tCU3F6S3diTUVvUk43Z3pNQWdZcG5DaWliZ1B2V240YStQMmxUWVh6eitMeGFwL2xFcTdpaVNvMHFaazlYU2FMTEtxUUkrWHZrbDBpcDFVPSIsIml2UGFyYW1ldGVyU3BlYyI6IjVlKzg0QWdReW1KTWhmZlQiLCJtYXRlcmlhbFNldFNlcmlhbCI6MX0%3D&branch=master)

[![BoltOps Badge](https://img.boltops.com/boltops/badges/boltops-badge.png)](https://www.boltops.com)

This configset installs, configure and runs awslogs.  This will send EC2 instance logs to [CloudWatch Logs](https://docs.aws.amazon.com/AmazonCloudWatch/latest/logs/WhatIsCloudWatchLogs.html).

## What are lono configsets?

Lono configsets allow CloudFormation [cfn-init](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/cfn-init.html) [configsets](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/aws-resource-init.html) that are typically embedded in the template to be reusable.  More info: [Lono Configsets docs](https://lono.cloud/docs/configsets/).

## Usage

Use `configset` to enable the configset for the blueprint.  Example:

configs/demo/configsets/base.rb:

```ruby
configset("amazon-linux-extras", resource: "Instance")
```

## Customize CloudWatch Log Group

By default, the cloudwatch log group will be `ec2`.  You can also customize whether the `{instance_id}` is added to the log_stream_name config as a prefix or a suffix. You can configure this with [Configset Variables](https://lono.cloud/docs/configsets/variables/).  Example:

configs/ec2/configsets/variables/awslogs.rb:

```ruby
@log_group_name = "my-log-group-name"
@add_instance_id_as = "suffix" # default is prefix
```

## Customize awslogs.conf

This configset creates a default `/etc/awslogs/awslogs.conf` with common logfiles to send to CloudWatch.  Here's a list of some of them from the `awslogs.conf` file.

    /var/log/boot.log
    /var/log/cfn-hup.log
    /var/log/cfn-init-cmd.log
    /var/log/cfn-init.log
    /var/log/cloud-init-output.log
    /var/log/cloud-init.log
    /var/log/cron
    /var/log/dmesg
    /var/log/messages

You can completely customize and configure your own `awslogs.conf` with the `@awslogs_conf` variable. Example:

configs/demo/configsets/variables/awslogs.rb:

```ruby
@awslogs_conf =<<EOL
[general]
state_file = /var/lib/awslogs/agent-state
[/var/log/amazon/ssm/amazon-ssm-agent.log]
datetime_format = %Y-%m-%d %H:%M:%S
file = /var/log/amazon/ssm/amazon-ssm-agent.log
log_stream_name = {instance_id}-/var/log/amazon/ssm/amazon-ssm-agent.log
log_group_name = my-log-group
[/var/log/amazon/ssm/errors.log]
datetime_format = %Y-%m-%d %H:%M:%S
file = /var/log/amazon/ssm/errors.log
log_stream_name = {instance_id}-/var/log/amazon/ssm/errors.log
log_group_name = my-log-group
EOL
```

This means the configset is quite reusable.

## IAM Role

Make sure that the EC2 instance IAM Role has permission to send logs to CloudWatch logs.  Otherwise the awslogs agent will fail to send the logs. You can debug this by logging into the server and looking at `/var/log/amazon/ssm/amazon-ssm-agent.log`.
