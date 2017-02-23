# zabbix-cloudwatch
Amazon CloudWatch is a monitoring service for AWS cloud resources and the applications you run on AWS, which can be accessed from AWS console. We have now integrated CloudWatch monitoring data into Zabbix monitoring system.

AmazonCloudWatch Developer Guide - http://docs.aws.amazon.com/AmazonCloudWatch/latest/DeveloperGuide/supported_services.html

AWS SDK in Python Boto - https://github.com/boto/boto

# Installation
Install boto on zabbix server via pip.
```
pip install boto
```

Clone this repo to /opt/zabbix
```
git clone https://github.com/omni-lchen/zabbix-cloudwatch.git cloudwatch
```

Customize `conf.d/crontab` to your settings where:

1. `<*_Name>` : The name in AWS
2. `<zabbix_host>` : The name of the host being monitored by Zabbix (not the Zabbix host)
3. `<zabbix_server or zabbix_proxy>` : Tho host running the Zabbix Server backend
4. `<aws_account>` : AWS Account number
5. `<aws_region>` : i.e. `us-east-1`

Customize `conf/awscreds` where:

1. `[aws_account_1]` : AWS Account number, i.e. `[<aws_account>]`
2. `aws_access_key_id` : Get it from the AWS website
3. `aws_secret_access_key` : Get it from the AWS website

Customize the metrics you want to capture in `conf/aws_services_metrics.conf`. Find the metrics from [AmazonCloudWatch Developer Guide](http://docs.aws.amazon.com/AmazonCloudWatch/latest/DeveloperGuide/supported_services.html).

Test that the cron script you are running works:
```
./cron.RDS.sh "cool-server" "mydb.abs343jddvzu.us-east-1.rds.amazonaws.com" "localhost" "12345678901" "us-east-1"
```

In the `cron.d` folder, run `crontab crontab` - ** WARNING this will clear your personal crontab! **

Make sure the cron job is being run with no errors in `syslog`.

Import `template\*` into Zabbix via the website. (It's as simple as it sounds)

Finally, associate an imported template with a host, which must match the name `<zabbix_host>`

### Advanced Help
Example AWS Metric Zabbix Trapper Item Key Format without Discovery.
```  
Key: \<aws_service\>.\<metric\>.\<statistics\>
```

Example AWS Metric Zabbix Trapper Item Key Format with Discovery.
```  
Key: \<aws_service\>.\<metric\>.\<statistics\>["\<aws_account\>","\<aws_region\>","\<discovery_item_value\>"]
```

DynamoDB Item Key Format is different from the standard setup, due to combinations of different dimensions and metrics.
```  
operations_all = ['GetItem', 'PutItem', 'Query', 'Scan', 'UpdateItem', 'DeleteItem', 'BatchGetItem', 'BatchWriteItem']
operations_metrics = ['SuccessfulRequestLatency', 'SystemErrors', 'ThrottledRequests]
```

DynamoDB Item Key1: `DynamoDB.\<operations_all\>.\<operations_metrics\>.\<statistics\>["\<aws_account\>","\<aws_region\>","\<table_name\>"]`
```
operations_returned_item = ['Query', 'Scan']
```  

DynamoDB Item Key2: `DynamoDB.\<operations_returned_item\>.ReturnedItemCount.\<statistics\>["\<aws_account\>","\<aws_region\>","\<table_name\>"]`

DynamoDB Other Keys: `DynamoDB.\<metric\>.\<statistics\>["\<aws_account\>","\<aws_region\>","\<table_name\>"]`
