# Backup Database S3

Terraform module for scheduled backing up of a single RDBMS (Postgresql or MySql) database, with notifications sent to a Slack webhook.

The container that the module uses (`digirati/backup-database-s3` - see https://github.com/digirati-co-uk/backup-database-s3) will attempt to load the specified engine family's Docker image that has been tagged with the engine version, and then use an engine family-specific command to create a gzip'd SQL dump of the reqiested database. The file will have the timestamp of the operation appended to its name and will be uploaded to the specified location in S3.

## Parameters

| Variable                        | Description                                                                                     | Default                            |
|---------------------------------|-------------------------------------------------------------------------------------------------|------------------------------------|
| prefix                          | Prefix to give to AWS resources                                                                 |                                    |
| engine_family                   | Identifier for the RDBMS family of Docker image to use - currently either `postgres` or `mysql` |                                    |
| engine_version                  | Numerical tag for RDBMS engine version e.g. `9.5.10`                                            |                                    |
| database_host                   | Host address of database server                                                                 |                                    |
| database_port                   | Database server port to use                                                                     |                                    |
| database_name                   | Name of database to back up                                                                     |                                    |
| database_username               | Username to authenticate to database with                                                       |                                    |
| database_password               | Password to authenticate to database with                                                       |                                    |
| slack_webhook_url               | Slack Webhook URL for notifications                                                             |                                    |
| log_group_name                  | CloudWatch log group name that the container will emit logs to                                  |                                    |
| backup_database_s3_docker_image | The Docker image to use for the ECS Task                                                        | digirati/backup-database-s3:latest |
| region                          | AWS Region for resources                                                                        |                                    |
| s3_key_prefix                   | The prefix for the S3 key to be used for backups                                                |                                    |
| s3_bucket_name                  | The name of the S3 bucket that will hold backups                                                |                                    |
| account_id                      | AWS account ID                                                                                  |                                    |
| cluster_id                      | The cluster on which to run the scheduled ECS task                                              |                                    |
| cron_expression                 | Cron scheduling expression in form `cron(x x x x x x)`                                          |                                    |

## Example

```
module "backup_presley" {
  source                          = "git::https://github.com/digirati-co-uk/terraform-backup-database-s3.git//"
  engine_family                   = "postgres"
  engine_version                  = "9.5.10"
  database_host                   = "${module.deliverator_db.address}"
  database_port                   = "${module.deliverator_db.port}"
  database_name                   = "${var.presley_database_name}"
  database_username               = "${var.presley_database_username}"
  database_password               = "${var.presley_database_password}"
  slack_webhook_url               = "${var.slack_webhook_status}"
  log_group_name                  = "${var.log_group_name}"
  prefix                          = "${var.prefix}"
  backup_identifier               = "presley"
  region                          = "${var.region}"
  s3_key_prefix                   = "backups/presley/"
  s3_bucket_name                  = "${var.bootstrap_objects_bucket}"
  account_id                      = "${var.account_id}"
  cluster_id                      = "${module.metropolis_cluster.cluster_id}"
  cron_expression                 = "cron(0 0 * * ? *)"
}

```

## File name format

The default filename format uses `date +%Y%m%d%H%M` to produce a timestamp.

For further information, see https://github.com/digirati-co-uk/backup-database-s3#s3_prefix

## Permissions

The following Terraform snippet shows the AWS permissions required for this module:

```
data "aws_iam_policy_document" "backup_bucket_access" {
  statement {
    effect = "Allow"

    actions = [
      "s3:PutObject",
    ]

    resources = [
      "${var.backup_bucket_arn}/${var.s3_key_prefix}*",
      "${var.backup_bucket_arn}/${var.s3_key_prefix}/*",
    ]
  }
}
```
