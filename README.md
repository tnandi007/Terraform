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
          type        = "AWS"
          identifiers = [var.trusted_role_arn]
        }
    
        principals {
          type        = "Federated"
          identifiers = ["arn:aws:iam::${var.account_id}:saml-provider/${var.provider_name}", "cognito-identity.amazonaws.com"]
        }
      }
    }
