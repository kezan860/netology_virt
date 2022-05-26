# Домашнее задание к занятию "7.3. Основы и принцип работы Терраформ"

## Задача 1. Создадим бэкэнд в S3 (необязательно, но крайне желательно).

Если в рамках предыдущего задания у вас уже есть аккаунт AWS, то давайте продолжим знакомство со взаимодействием
терраформа и aws. 

1. Создайте s3 бакет, iam роль и пользователя от которого будет работать терраформ. Можно создать отдельного пользователя,
а можно использовать созданного в рамках предыдущего задания, просто добавьте ему необходимы права, как описано 
[здесь](https://www.terraform.io/docs/backends/types/s3.html).
2. Зарегистрируйте бэкэнд в терраформ проекте как описано по ссылке выше. 


## Задача 2. Инициализируем проект и создаем воркспейсы. 

1. Выполните `terraform init`:
    * если был создан бэкэнд в S3, то терраформ создат файл стейтов в S3 и запись в таблице 
dynamodb.
    * иначе будет создан локальный файл со стейтами.  
2. Создайте два воркспейса `stage` и `prod`.
3. В уже созданный `aws_instance` добавьте зависимость типа инстанса от вокспейса, что бы в разных ворскспейсах 
использовались разные `instance_type`.
4. Добавим `count`. Для `stage` должен создаться один экземпляр `ec2`, а для `prod` два. 
5. Создайте рядом еще один `aws_instance`, но теперь определите их количество при помощи `for_each`, а не `count`.
6. Что бы при изменении типа инстанса не возникло ситуации, когда не будет ни одного инстанса добавьте параметр
жизненного цикла `create_before_destroy = true` в один из рессурсов `aws_instance`.
7. При желании поэкспериментируйте с другими параметрами и рессурсами.

В виде результата работы пришлите:
* Вывод команды `terraform workspace list`.
* Вывод команды `terraform plan` для воркспейса `prod`.  

## Ответ:

```
✗ terraform init


✗ terraform workspace new stage
Created and switched to workspace "stage"!

You're now on a new, empty workspace. Workspaces isolate their state,
so if you run "terraform plan" Terraform will not see any existing state
for this configuration.


✗ terraform workspace new prod
Created and switched to workspace "prod"!

You're now on a new, empty workspace. Workspaces isolate their state,
so if you run "terraform plan" Terraform will not see any existing state
for this configuration.


✗ terraform workspace list
  default
  stage
* prod


Переключение на прод:
✗ terraform workspace select prod
Switched to workspace "prod".


Содержимое s3.tf:

        provider "aws" {
                region = "us-west-2"
        }
        resource "aws_s3_bucket" "bucket" {
          bucket = "netology-bucket-${terraform.workspace}"
          acl    = "private"
          tags = {
            Name        = "Bucket1"
            Environment = terraform.workspace
          }
        }


Вывод Комманд:
terraform init
        
        Initializing the backend...
        
        Initializing provider plugins...
        - Reusing previous version of hashicorp/aws from the dependency lock file
        - Using previously-installed hashicorp/aws v3.32.0
        
        Terraform has been successfully initialized!
        
        You may now begin working with Terraform. Try running "terraform plan" to see
        any changes that are required for your infrastructure. All Terraform commands
        should now work.
        
        If you ever set or change modules or backend configuration for Terraform,
        rerun this command to reinitialize your working directory. If you forget, other
        commands will detect it and remind you to do so if necessary.



terraform plan
        
        An execution plan has been generated and is shown below.
        Resource actions are indicated with the following symbols:
          + create
        
        Terraform will perform the following actions:
        
          # aws_s3_bucket.bucket will be created
          + resource "aws_s3_bucket" "bucket" {
              + acceleration_status         = (known after apply)
              + acl                         = "private"
              + arn                         = (known after apply)
              + bucket                      = "netology-bucket-prod"
              + bucket_domain_name          = (known after apply)
              + bucket_regional_domain_name = (known after apply)
              + force_destroy               = false
              + hosted_zone_id              = (known after apply)
              + id                          = (known after apply)
              + region                      = (known after apply)
              + request_payer               = (known after apply)
              + tags                        = {
                  + "Environment" = "prod"
                  + "Name"        = "Bucket1"
                }
              + website_domain              = (known after apply)
              + website_endpoint            = (known after apply)
        
              + versioning {
                  + enabled    = (known after apply)
                  + mfa_delete = (known after apply)
                }
            }
        
        Plan: 1 to add, 0 to change, 0 to destroy.
        
        ------------------------------------------------------------------------
        
        Note: You didn't specify an "-out" parameter to save this plan, so Terraform
        can't guarantee that exactly these actions will be performed if
        "terraform apply" is subsequently run.
        
        12:19:22 alex@upc(0):~/devops-netology/terraform/s3$ terraform workspace list
          default
          dev
        * prod
          stage
        
        12:19:35 alex@upc(0):~/devops-netology/terraform/s3$ terraform apply
        
        An execution plan has been generated and is shown below.
        Resource actions are indicated with the following symbols:
          + create
        
        Terraform will perform the following actions:
        
          # aws_s3_bucket.bucket will be created
          + resource "aws_s3_bucket" "bucket" {
              + acceleration_status         = (known after apply)
              + acl                         = "private"
              + arn                         = (known after apply)
              + bucket                      = "netology-bucket-prod"
              + bucket_domain_name          = (known after apply)
              + bucket_regional_domain_name = (known after apply)
              + force_destroy               = false
              + hosted_zone_id              = (known after apply)
              + id                          = (known after apply)
              + region                      = (known after apply)
              + request_payer               = (known after apply)
              + tags                        = {
                  + "Environment" = "prod"
                  + "Name"        = "Bucket1"
                }
              + website_domain              = (known after apply)
              + website_endpoint            = (known after apply)
        
              + versioning {
                  + enabled    = (known after apply)
                  + mfa_delete = (known after apply)
                }
            }
        
        Plan: 1 to add, 0 to change, 0 to destroy.
        
        Do you want to perform these actions in workspace "prod"?
          Terraform will perform the actions described above.
          Only 'yes' will be accepted to approve.
        
          Enter a value: yes
        
        aws_s3_bucket.bucket: Creating...
        aws_s3_bucket.bucket: Still creating... [10s elapsed]
        aws_s3_bucket.bucket: Still creating... [20s elapsed]
        aws_s3_bucket.bucket: Creation complete after 21s [id=netology-bucket-prod]



Содержимое s3.tf:

provider "aws" {
        region = "us-west-2"
}

locals {
  web_instance_count_map = {
  stage = 1
  prod = 2
  }
}

resource "aws_s3_bucket" "bucket" {
  bucket = "netology-bucket-${count.index}-${terraform.workspace}"
  acl    = "private"
  tags = {
    Name        = "Bucket ${count.index}"
    Environment = terraform.workspace
  }
  count = local.web_instance_count_map[terraform.workspace]
}



Содержимое s3.tf:

provider "aws" {
        region = "us-west-2"
}

locals {
  web_instance_count_map = {
  stage = 1
  prod = 2
  }
}

resource "aws_s3_bucket" "bucket" {
  bucket = "netology-bucket-${count.index}-${terraform.workspace}"
  acl    = "private"
  tags = {
    Name        = "Bucket ${count.index}"
    Environment = terraform.workspace
  }
  count = local.web_instance_count_map[terraform.workspace]
}

locals {
  backets_ids = toset([
    "e1",
    "e2",
  ])
}
resource "aws_s3_bucket" "bucket_e" {
  for_each = local.backets_ids
  bucket = "netology-bucket-${each.key}-${terraform.workspace}"
  acl    = "private"
  tags = {
    Name        = "Bucket ${each.key}"
    Environment = terraform.workspace
  }
}
```
