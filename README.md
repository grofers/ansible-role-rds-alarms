[![Build Status](https://travis-ci.org/grofers/ansible-role-rds-alarms.svg?branch=master)](https://travis-ci.org/grofers/ansible-role-rds-alarms)

# RDS Alarms

Creates warning and critical alarms for RDS instances on Amazon CloudWatch. For
more details, check out the [blog post](https://lambda.grofers.com/2017/02/28/ansible-at-grofers-part-2-managing-postgresql/).

:boom: Battle-tested at [Grofers](https://grofers.com)

## Requirements

* A topic in [AWS SNS](https://aws.amazon.com/sns/)
* [boto](https://pypi.python.org/pypi/boto/)
* [awscli](https://aws.amazon.com/cli/)
* An IAM Policy with the following permissions:
```json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Sid": "Stmt19471460522000",
            "Effect": "Allow",
            "Action": [
                "cloudwatch:DeleteAlarms",
                "cloudwatch:DescribeAlarms",
                "cloudwatch:PutMetricAlarm"
            ],
            "Resource": [
                "*"
            ]
        },
        {
            "Sid": "Stmt1947940274000",
            "Effect": "Allow",
            "Action": [
                "rds:DescribeDBInstances"
            ],
            "Resource": [
                "*"
            ]
        }
    ]
}
```

## Installation

To install, just run
```shell
$ ansible-galaxy install grofers.rds-alarms
```

## Role Variables

* `rds_alarms_region` - AWS region (Required)
* `rds_alarms_common_action_list` - List of ARN of topics in AWS SNS
* `rds_alarms_period` - Time (in seconds) between metric evaluations
* `rds_alarms_evaluation_periods` - The number of times in which the
metric is evaluated before final calculation
* `rds_alarms_common_action_list` - Always include this alarm actions
* `rds_alarms_warning_threshold` - Threshold for warning (default - 75%)
* `rds_alarms_critical_threshold` - Threshold for warning (default - 90%)
* `rds_alarms_warning_cpu_credits_threshold` - Threshold for CPU Credits
(default - 30)
* `rds_alarms_critical_cpu_credits_threshold` - Threshold for CPU Credits
(default - 15)
* `rds_alarms_db_instances` - Dict with the following format:
```yaml
rds_alarms_db_instances:
  <rds-instance-identifier>:
    warning_db_connections_threshold: 100
    critical_db_connections_threshold: 200
    alarm_action_list: ["arn:aws:sns:us-east-1:9783248248:MYALARM"]
    critical_threshold: 90 # Optional
    warning_threshold: 75 # Optional
    credit_warning_threshold: 30 # Optional
    credit_critical_threshold: 15 # Optional
    replica_lag_threshold: 1800 # Required Only for replicas. Units seconds
```

## Conventions

The format of name of alarms created in Amazon CloudWatch is:
`rds-<instance_name>-<metric_name>-<alert_type>`.

For example, `warning` alarm for `CPU` for an instance with identifier
`my-rds-instance` will be created as `rds-my-rds-instance-cpu-warning`.

## Example Playbook

This playbook will create alarms for `my-rds-instance-identifier` with the
default thresholds. Where as for alarms created for
`my-replica-rds-instance-identifier` will be created with warning threshold of
80% and critical threshold will be the default value(90%). If the instance is a
replica then an alarm will also be created for the replica lag. For every t2
instance, alarms are also created for remaining CPU credits.

```yaml
- hosts: localhost
  connection: local
  vars:
    rds_alarms_common_action_list:
      - "arn:aws:sns:us-east-1:9783248248:ALARMS"
    rds_alarms_period: 60
    rds_alarms_evaluation_periods: 2
    rds_alarms_region: us-east-1
    rds_alarms_warning_threshold: 70
    rds_alarms_critical_threshold: 90
    rds_alarms_warning_cpu_credits_threshold: 60
    rds_alarms_critical_cpu_credits_threshold: 30
    rds_alarms_db_instances:
      my-rds-instance-identifier: # this will use the defaults
        warning_db_connections_threshold: 100
        critical_db_connections_threshold: 200
        alarm_action_list: ["arn:aws:sns:us-east-1:9783248248:MYALARM"]
      my-replica-rds-instance-identifier:
        warning_threshold: 80
        warning_db_connections_threshold: 100
        critical_db_connections_threshold: 200
        alarm_action_list: ["arn:aws:sns:us-east-1:9783248248:MYALARM"]
        credit_warning_threshold: 20
        credit_critical_threshold: 10
        replica_lag_threshold: 1800
  roles:
    - rds-alarms

```


## Limitations

You need to create multiple playbooks for different regions.

## License

[MIT License](LICENSE)
