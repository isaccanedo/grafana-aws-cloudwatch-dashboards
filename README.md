[<img src="https://monitoringartist.github.io/managed-by-monitoringartist.png" alt="Managed by Monitoring Artist: DevOps / Docker / Kubernetes / AWS ECS / Zabbix / Zenoss / Terraform / Monitoring" align="right"/>](http://www.monitoringartist.com 'DevOps / Docker / Kubernetes / AWS ECS / Zabbix / Zenoss / Terraform / Monitoring')

# Painéis Grafana para AWS CloudWatch

Conjunto de painéis do AWS Grafana publicados em
[grafana.com](https://grafana.com/dashboards?dataSource=cloudwatch) -
Mais de 10 milhões de downloads.

Doc:
- [Configuração da fonte de dados Cloudwatch](https://grafana.com/docs/grafana/latest/datasources/cloudwatch/)
- [Grafana doc](https://grafana.com/docs/)

Sinta-se à vontade para criar pull request para recursos/printscreens/...

Defina as variáveis do seu painel (`Região, ...`) após a importação do painel.
As variáveis vazias do painel são o motivo do erro inicial *"Não foi possível chamar a API da AWS" ou "Erro de solicitação de métrica"*.

Importe todos os painéis AWS do Monitoring Artist de uma só vez (script de exemplo,
`bash/curl/jq` required):

```bash
#!/bin/bash
jq --version >/dev/null 2>&1 || { echo >&2 "I require jq but it's not installed. Aborting."; exit 1; }
### Edite as variáveis grafana_* para corresponder à sua configuração do Grafana:
grafana_host="http://localhost:3000"
grafana_cred="admin:admin"
# Mantenha grafana_folder vazio para adicionar os dashboards na pasta "Geral"
grafana_folder="AWS CloudWatch"
ds=(1516 677 139 674 590 659 758 623 617 551 653 969 650 644 607 593 707 575 1519 581 584 2969 8050 11099 11154 11155 12979 13018 13040 13104 13892 14189 14391 14392 14954 14955 15016 16899 17741 17978);
folderId=$(curl -s -k -u "$grafana_cred" $grafana_host/api/folders | jq -r --arg grafana_folder  "$grafana_folder" '.[] | select(.title==$grafana_folder).id')
if [ -z "$folderId" ] ; then echo "Didn't get folderId" ; else echo "Got folderId $folderId" ; fi
for d in "${ds[@]}"; do
  echo -n "Processing $d: "
  j=$(curl -s -k -u "$grafana_cred" $grafana_host/api/gnet/dashboards/$d | jq .json)
  payload="{\"dashboard\":$j,\"overwrite\":true"
  if [ ! -z "$folderId" ] ; then payload="${payload}, \"folderId\": $folderId }";  else payload="${payload} }" ; fi
  curl -s -k -u "$grafana_cred" -XPOST -H "Accept: application/json" \
    -H "Content-Type: application/json" \
    -d "$payload" \
    $grafana_host/api/dashboards/import; echo ""
done
```

Use o [AWS Policy Generator](http://awspolicygen.s3.amazonaws.com/policygen.html),
que se adapta às suas necessidades. Exemplo de função IAM mínima para Grafana (CloudWatch + métricas EC2):

```
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Sid": "AllowReadingMetricsFromCloudWatch",
            "Effect": "Allow",
            "Action": [
                "cloudwatch:DescribeAlarmsForMetric",
                "cloudwatch:DescribeAlarmHistory",
                "cloudwatch:DescribeAlarms",
                "cloudwatch:ListMetrics",
                "cloudwatch:GetMetricStatistics",
                "cloudwatch:GetMetricData"
            ],
            "Resource": "*"
        },
        {
            "Sid": "AllowReadingTagsInstancesRegionsFromEC2",
            "Effect": "Allow",
            "Action": [
                "ec2:DescribeTags",
                "ec2:DescribeInstances",
                "ec2:DescribeRegions"
            ],
            "Resource": "*"
        },
        {
            "Sid": "AllowReadingResourcesForTags",
            "Effect" : "Allow",
            "Action" : "tag:GetResources",
            "Resource" : "*"
        }
    ]
}
```
Consulte https://github.com/monitoringartist/grafana-cross-account-cloudwatch-access para acesso entre contas da AWS.

Você também pode instalar este projeto como uma biblioteca Jsonnet com [jsonnet-bundler](https://github.com/jsonnet-bundler/jsonnet-bundler):

```shell
$ jb install github.com/monitoringartist/grafana-aws-cloudwatch-dashboards
$ cat > aws-cloudwatch-dashboards.jsonnet <<EOF
local awsCloudWatch = import 'github.com/monitoringartist/grafana-aws-cloudwatch-dashboards/dashboards.libsonnet';

awsCloudWatch.grafanaDashboards
EOF
$ jsonnet -J vendor aws-cloudwatch-dashboards.jsonnet
```

Provisionamento de clique único [![Gitpod ready-to-test](https://img.shields.io/badge/Gitpod-ready--to--test-blue?logo=gitpod)](https://gitpod.io/#https://github.com/monitoringartist/grafana-aws-cloudwatch-dashboards/) - faça login como `admin/admin` e crie a fonte de dados CloudWatch a partir de sua `chave de acesso e segredo` para ver todos os painéis com seus dados.

### [AWS API Gateway](https://github.com/monitoringartist/grafana-aws-cloudwatch-dashboards/tree/master/aws-api-gateway)
[![AWS API Gateway](aws-api-gateway/aws-api-gateway.png)](https://github.com/monitoringartist/grafana-aws-cloudwatch-dashboards/tree/master/aws-api-gateway)

### [AWS Auto Scaling](https://github.com/monitoringartist/grafana-aws-cloudwatch-dashboards/tree/master/aws-autoscaling)

### [AWS Billing](https://github.com/monitoringartist/grafana-aws-cloudwatch-dashboards/tree/master/aws-billing)
[![AWS Billing](aws-billing/aws-billing.png)](https://github.com/monitoringartist/grafana-aws-cloudwatch-dashboards/tree/master/aws-billing)

### [AWS Certificate Manager](https://github.com/monitoringartist/grafana-aws-cloudwatch-dashboards/tree/master/aws-certificate-manager)
[![AWS Certificate Manager](aws-certificate-manager/aws-certificate-manager.png)](https://github.com/monitoringartist/grafana-aws-cloudwatch-dashboards/tree/master/aws-certificate-manager)

### [AWS CloudFront](https://github.com/monitoringartist/grafana-aws-cloudwatch-dashboards/tree/master/aws-cloudfront)
[![AWS CloudFront](aws-cloudfront/aws-cloudfront.png)](https://github.com/monitoringartist/grafana-aws-cloudwatch-dashboards/tree/master/aws-cloudfront)

### [AWS CloudWatch Browser](https://github.com/monitoringartist/grafana-aws-cloudwatch-dashboards/tree/master/aws-cloudwatch-browser)
[![AWS Cloudwatch Browser](aws-cloudwatch-browser/aws-cloudwatch-browser.png)](https://github.com/monitoringartist/grafana-aws-cloudwatch-dashboards/tree/master/aws-cloudwatch-browser)

### [AWS CloudWatch Synthetics](https://github.com/monitoringartist/grafana-aws-cloudwatch-dashboards/tree/master/aws-cloudwatch-synthetics)
[![AWS Cloudwatch Synthetics](aws-cloudwatch-synthetics/aws-cloudwatch-synthetics.png)](https://github.com/monitoringartist/grafana-aws-cloudwatch-dashboards/tree/master/aws-cloudwatch-synthetics)

### [AWS CloudWatch Usage Metrics](https://github.com/monitoringartist/grafana-aws-cloudwatch-dashboards/tree/master/aws-cloudwatch-usage-metrics)
[![AWS Cloudwatch Browser](aws-cloudwatch-usage-metrics/aws-cloudwatch-usage-metrics.png)](https://github.com/monitoringartist/grafana-aws-cloudwatch-dashboards/tree/master/aws-cloudwatch-usage-metrics)

### [AWS CodeBuild](https://github.com/monitoringartist/grafana-aws-cloudwatch-dashboards/tree/master/aws-codebuild)
[![AWS CodeBuild](aws-codebuild/aws-codebuild.png)](https://github.com/monitoringartist/grafana-aws-cloudwatch-dashboards/tree/master/aws-codebuild)

### [AWS Cognito](https://github.com/monitoringartist/grafana-aws-cloudwatch-dashboards/tree/master/aws-cognito)

### [AWS Direct Connect](https://github.com/monitoringartist/grafana-aws-cloudwatch-dashboards/tree/master/aws-direct-connect)
[![AWS Direct Connect](aws-direct-connect/aws-direct-connect.png)](https://github.com/monitoringartist/grafana-aws-cloudwatch-dashboards/tree/master/aws-direct-connect)

### [AWS DynamoDB](https://github.com/monitoringartist/grafana-aws-cloudwatch-dashboards/tree/master/aws-dynamodb)

### [AWS EBS](https://github.com/monitoringartist/grafana-aws-cloudwatch-dashboards/tree/master/aws-ebs)
[![AWS EBS](aws-ebs/aws-ebs.png)](https://github.com/monitoringartist/grafana-aws-cloudwatch-dashboards/tree/master/aws-ebs)

### [AWS EC2](https://github.com/monitoringartist/grafana-aws-cloudwatch-dashboards/tree/master/aws-ec2)
[![AWS EC2](aws-ec2/aws-ec2.png)](https://github.com/monitoringartist/grafana-aws-cloudwatch-dashboards/tree/master/aws-ec2)

### [AWS ECS](https://github.com/monitoringartist/grafana-aws-cloudwatch-dashboards/tree/master/aws-ecs)
[![AWS ECS](aws-ecs/aws-ecs.png)](https://github.com/monitoringartist/grafana-aws-cloudwatch-dashboards/tree/master/aws-ecs)

### [AWS EFS](https://github.com/monitoringartist/grafana-aws-cloudwatch-dashboards/tree/master/aws-efs)
[![AWS EFS](aws-efs/aws-efs.png)](https://github.com/monitoringartist/grafana-aws-cloudwatch-dashboards/tree/master/aws-efs)

### [AWS EKS](https://github.com/monitoringartist/grafana-aws-cloudwatch-dashboards/tree/master/aws-eks)

### [AWS ElastiCache Redis](https://github.com/monitoringartist/grafana-aws-cloudwatch-dashboards/tree/master/aws-elasticache)
[![AWS ElastiCache Redis](aws-elasticache/aws-elasticache-redis.png)](https://github.com/monitoringartist/grafana-aws-cloudwatch-dashboards/tree/master/aws-elasticache)

### [AWS ELB Classic Load Balancer](https://github.com/monitoringartist/grafana-aws-cloudwatch-dashboards/tree/master/aws-elb)
[![AWS ELB Classic Load Balancer](aws-elb/aws-elb-classic-lb.png)](https://github.com/monitoringartist/grafana-aws-cloudwatch-dashboards/tree/master/aws-elb)

### [AWS ELB Application Load Balancer](https://github.com/monitoringartist/grafana-aws-cloudwatch-dashboards/tree/master/aws-elb)
[![AWS ELB Application Load Balancer](aws-elb/aws-elb-application-lb.png)](https://github.com/monitoringartist/grafana-aws-cloudwatch-dashboards/tree/master/aws-elb)

### [AWS EMR Hadoop 2](https://github.com/monitoringartist/grafana-aws-cloudwatch-dashboards/tree/master/aws-emr)
[![AWS EMR Hadoop 2](aws-emr/aws-emr-hadoop-2.png)](https://github.com/monitoringartist/grafana-aws-cloudwatch-dashboards/tree/master/aws-emr)

### [AWS Eventbridge](https://github.com/monitoringartist/grafana-aws-cloudwatch-dashboards/tree/master/aws-eventbridge)
[![AWS Eventbridge](aws-eventbridge/aws-eventbridge.png)](https://github.com/monitoringartist/grafana-aws-cloudwatch-dashboards/tree/master/aws-eventbridge)

### [AWS Inspector](https://github.com/monitoringartist/grafana-aws-cloudwatch-dashboards/tree/master/aws-inspector)
[![AWS Inspector](aws-inspector/aws-inspector.png)](https://github.com/monitoringartist/grafana-aws-cloudwatch-dashboards/tree/master/aws-inspector)

### [AWS Kinesis](https://github.com/monitoringartist/grafana-aws-cloudwatch-dashboards/tree/master/aws-kinesis)
[![AWS Kinesis](aws-kinesis/aws-kinesis.png)](https://github.com/monitoringartist/grafana-aws-cloudwatch-dashboards/tree/master/aws-kinesis)

### [AWS Lambda](https://github.com/monitoringartist/grafana-aws-cloudwatch-dashboards/tree/master/aws-lambda)
[![AWS Lambda](aws-lambda/aws-lambda.png)](https://github.com/monitoringartist/grafana-aws-cloudwatch-dashboards/tree/master/aws-lambda)

### [AWS Logs](https://github.com/monitoringartist/grafana-aws-cloudwatch-dashboards/tree/master/aws-logs)
[![AWS Logs](aws-logs/aws-logs.png)](https://github.com/monitoringartist/grafana-aws-cloudwatch-dashboards/tree/master/aws-logs)

### [AWS Network Firewall](https://github.com/monitoringartist/grafana-aws-cloudwatch-dashboards/tree/master/aws-network-firewall)
[![AWS Network Firewall](aws-network-firewall/aws-network-firewall.png)](https://github.com/monitoringartist/grafana-aws-cloudwatch-dashboards/tree/master/aws-network-firewall)

### [AWS Prometheus](https://github.com/monitoringartist/grafana-aws-cloudwatch-dashboards/tree/master/aws-prometheus)
[![AWS Prometheus](aws-prometheus/aws-prometheus.png)](https://github.com/monitoringartist/grafana-aws-cloudwatch-dashboards/tree/master/aws-prometheus)

### [AWS RDS](https://github.com/monitoringartist/grafana-aws-cloudwatch-dashboards/tree/master/aws-rds)
[![AWS RDS](aws-rds/aws-rds.png)](https://github.com/monitoringartist/grafana-aws-cloudwatch-dashboards/tree/master/aws-rds)

### [AWS Redshift](https://github.com/monitoringartist/grafana-aws-cloudwatch-dashboards/tree/master/aws-redshift)
[![AWS Redshift](aws-redshift/aws-redshift.png)](https://github.com/monitoringartist/grafana-aws-cloudwatch-dashboards/tree/master/aws-redshift)

### [AWS Route 53](https://github.com/monitoringartist/grafana-aws-cloudwatch-dashboards/tree/master/aws-route-53)

### [AWS S3](https://github.com/monitoringartist/grafana-aws-cloudwatch-dashboards/tree/master/aws-s3)
[![AWS S3](aws-s3/aws-s3.png)](https://github.com/monitoringartist/grafana-aws-cloudwatch-dashboards/tree/master/aws-s3)

### [AWS SES](https://github.com/monitoringartist/grafana-aws-cloudwatch-dashboards/tree/master/aws-ses)

### [AWS SNS](https://github.com/monitoringartist/grafana-aws-cloudwatch-dashboards/tree/master/aws-sns)
[![AWS SNS](aws-sns/aws-sns.png)](https://github.com/monitoringartist/grafana-aws-cloudwatch-dashboards/tree/master/aws-sns)

### [AWS SQS](https://github.com/monitoringartist/grafana-aws-cloudwatch-dashboards/tree/master/aws-sqs)
[![AWS SQS](aws-sqs/aws-sqs.png)](https://github.com/monitoringartist/grafana-aws-cloudwatch-dashboards/tree/master/aws-sqs)

### [AWS Step Functions](https://github.com/monitoringartist/grafana-aws-cloudwatch-dashboards/tree/master/aws-step-functions)
[![AWS SNS](aws-step-functions/aws-step-functions.png)](https://github.com/monitoringartist/grafana-aws-cloudwatch-dashboards/tree/master/aws-step-functions)

### [AWS Storage Gateway](https://github.com/monitoringartist/grafana-aws-cloudwatch-dashboards/tree/master/aws-storage-gateway)
[![AWS Storage Gateway](aws-storage-gateway/aws-storage-gateway.png)](https://github.com/monitoringartist/grafana-aws-cloudwatch-dashboards/tree/master/aws-storage-gateway)

### [AWS Transit Gateway](https://github.com/monitoringartist/grafana-aws-cloudwatch-dashboards/tree/master/aws-transit-gateway)
[![AWS Transit Gateway](aws-transit-gateway/aws-transit-gateway.png)](https://github.com/monitoringartist/grafana-aws-cloudwatch-dashboards/tree/master/aws-transit-gateway)

### [AWS VPN](https://github.com/monitoringartist/grafana-aws-cloudwatch-dashboards/tree/master/aws-vpn)

### [AWS X-Ray](https://github.com/monitoringartist/grafana-aws-cloudwatch-dashboards/tree/master/aws-x-ray)
[![AWS X-Ray](aws-x-ray/aws-x-ray.png)](https://github.com/monitoringartist/grafana-aws-cloudwatch-dashboards/tree/master/aws-x-ray)

# Author

[Devops Monitoring Expert](http://www.jangaraj.com 'DevOps / Docker / Kubernetes / AWS ECS / Google GCP / Zabbix / Zenoss / Terraform / Monitoring'),
que adora sistemas de monitoramento e tecnologias de ponta: Docker,
Kubernetes, ECS, AWS, Google GCP, Terraform, Lambda, Zabbix, Grafana, Elasticsearch,
Kibana, Prometeu, Sysdig,...

Sumário:
* 4 000+ [GitHub](https://github.com/monitoringartist/) stars
* 10 000 000+ [Grafana dashboard](https://grafana.net/monitoringartist) downloads
* 60 000 000+ [Docker images](https://hub.docker.com/u/monitoringartist/) downloads

Professional devops / monitoring / consulting services:

[![Monitoring Artist](http://monitoringartist.com/img/github-monitoring-artist-logo.jpg)](http://www.monitoringartist.com 'DevOps / Docker / Kubernetes / AWS ECS / Google GCP / Zabbix / Zenoss / Terraform / Monitoring')
