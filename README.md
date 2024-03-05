# Using Dynamic Variable in Terraform
Lets assume we have to create a policy document like this 
    
    data "aws_iam_policy_document" "event_stream_bucket_role_assume_role_policy" {
   
    statement {
        actions = ["sts:AssumeRole"]
        
        principals {
          type        = "Service"
          identifiers = ["firehose.amazonaws.com"]
        }  

         principals {
          type        = "Service"
          identifiers = ["firehose.amazonaws.com"]
        }  

        actions = [
          "kms:Decrypt",
          "kms:GenerateDataKey"
            ]
    
        resources = ["*"]
    
        condition {
          test     = "ForAnyValue:StringEquals"
          variable = "kms:EncryptionContext:service"
          values   = ["pi"]
        }
      }      
    }
This can be implemented using Dynamic like this

        data "aws_iam_policy_document" "this" {
        statement {
              dynamic "principals" {
                for_each = var.policy.principals
                content {
                  type= principals.value["type"]
                  identifiers = principals.value["identifiers"]
                }
              }

              actions = var.policy.actions
        
              resources=[
                 "arn:aws:s3:::${local.bucket_name}/*", 
                 "arn:aws:s3:::${local.bucket_name}",
              ]
              dynamic "condition" {
                for_each = var.policy.condition
                content {        
                  test     = condition.value["test"]
                  variable = condition.value["variable"]
                  values   = condition.value["values"]
                }
              }
            }
    
          }

The variabke "Policy" has beed defined like following 

    variable "policy"{
      type =object({ 
          actions = list (string)
          #resorces = list (string) 
          principals = list (object({
            type= string
            identifiers = list(string)
          }))
          condition = list (object({
            test     = string
            variable = string
            values   = list(string)
          }))         
        })    
    }
    
In the main.tf how this moduled would be called ? like following

    module "s3-data-bucket" {
      source = "./modules/dataBucket"
     
      policy = {
         actions = [ "s3:*"]
          principals = [
              {
                identifiers = [ "arn:aws:iam::${local.corp_cd_account_id}:user/jenkins", "arn:aws:iam::${local.corp_cd_account_id}:root", "arn:aws:iam::${local.corp_cd_prod_account_id}:role/dp-jenkins-eks-worker-role"]
                type = "AWS"
              },
              {
                identifiers = [ "cloudfront.amazonaws.com" ]
                type = "Service"
              }         
            ]
          condition = []
        }  
    }

