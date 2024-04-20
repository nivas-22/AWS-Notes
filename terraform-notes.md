## Table of Content[](https://jhooq.com/terraform-secure-sensitive-data/#table-of-content)

1. [**Introduction**](https://jhooq.com/terraform-secure-sensitive-data/#1-introduction)
2. [**Securing Sensitive Data with Terraform**](https://jhooq.com/terraform-secure-sensitive-data/#2-securing-sensitive-data-with-terraform)
    - 2.1 [**Securely managing Terraform variables**](https://jhooq.com/terraform-secure-sensitive-data/#21-securely-managing-terraform-variables)
    - 2.2 [**Encrypting Sensitive Data at Rest**](https://jhooq.com/terraform-secure-sensitive-data/#22-encrypting-sensitive-data-at-rest)
    - 2.3 [**Encrypting Sensitive Data in Transit**](https://jhooq.com/terraform-secure-sensitive-data/#23-encrypting-sensitive-data-in-transit)
    - 2.4 [**The use of sensitive argument in Terraform to prevent logging of sensitive data**](https://jhooq.com/terraform-secure-sensitive-data/#24-the-use-of-sensitive-argument-in-terraform-to-prevent-logging-of-sensitive-data)
3. [**Practical Examples of Securing Sensitive Data**](https://jhooq.com/terraform-secure-sensitive-data/#3-practical-examples-of-securing-sensitive-data)
    - 3.1 [**Using environment variables for sensitive data**](https://jhooq.com/terraform-secure-sensitive-data/#31-using-environment-variables-for-sensitive-data)
    - 3.2 [**Securing sensitive data with AWS Key Management Service (KMS)**](https://jhooq.com/terraform-secure-sensitive-data/#32-securing-sensitive-data-with-aws-key-management-service-kms)
    - 3.3 [**Implementing Vault for secrets management**](https://jhooq.com/terraform-secure-sensitive-data/#33-implementing-vault-for-secrets-management)
4. [**Regular Auditing and Rotating Secrets**](https://jhooq.com/terraform-secure-sensitive-data/#4-regular-auditing-and-rotating-secrets)
    - 4.1 [**The importance of auditing**](https://jhooq.com/terraform-secure-sensitive-data/#41-the-importance-of-auditing)
    - 4.2 [**Regular rotation of secrets**](https://jhooq.com/terraform-secure-sensitive-data/#42-regular-rotation-of-secrets)
5. [**Conclusion**](https://jhooq.com/terraform-secure-sensitive-data/#5-conclusion)

  

## 1. Introduction[](https://jhooq.com/terraform-secure-sensitive-data/#1-introduction)

**Sensitive data?** **Yes**, you heard it right. We're talking about the data that you don't want to expose to the public. This might be your -

1. Database Credentials
2. SSH keys
3. API tokens -

Anything that could cause a significant security risk if it fell into the wrong hands. Handling this data properly is an essential part of our work in a [Terraform](https://www.terraform.io/) environment.

  

## 2. Securing Sensitive Data with Terraform[](https://jhooq.com/terraform-secure-sensitive-data/#2-securing-sensitive-data-with-terraform)

Handling sensitive data properly can be tricky, but don't worry, Terraform has got you covered! Here are some of the best practices that you should consider:

### 2.1 Securely managing Terraform variables[](https://jhooq.com/terraform-secure-sensitive-data/#21-securely-managing-terraform-variables)

This is your first line of defense. Avoid **hardcoding** sensitive data into your configuration files. Terraform variables should be your go-to choice for handling such data.

```terraform
# Set sensitive flag=true
```

```terraform
variable "password" {
```

```terraform
   description = "Database password"
```

```terraform
   type        = string
```

```terraform
   sensitive   = true
```

```terraform
} 
```

[](https://jhooq.com/terraform-secure-sensitive-data/# "Copy Code")[](https://jhooq.com/terraform-secure-sensitive-data/# "Toggle Line Numbers")

terraform

  

### 2.2 Encrypting Sensitive Data at Rest[](https://jhooq.com/terraform-secure-sensitive-data/#22-encrypting-sensitive-data-at-rest)

One of the most common places to store sensitive data at rest is in an **[AWS S3 bucket](https://aws.amazon.com/s3/)**.

> **But how do we ensure that our data is encrypted in S3 Bucket?**

AWS offers a solution called **Server-Side Encryption** with **[Amazon S3-Managed Keys (SSE-S3)](https://docs.aws.amazon.com/AmazonS3/latest/userguide/UsingServerSideEncryption.html)**.

When you enable **SSE-S3**, each object in your bucket is encrypted with a unique key. As an additional safeguard, this key itself is encrypted with a master key that Amazon regularly rotates.

Here is an Example Terraform code which you could refer:

```terraform
# Enable encryption SSE-S3 for S3 Bucket
```

```terraform
resource "aws_s3_bucket" "bucket" {
```

```terraform
  bucket = "my_tfstate_bucket"
```

```terraform
  acl    = "private"
```

```terraform
  server_side_encryption_configuration {
```

```terraform
    rule {
```

```terraform
      apply_server_side_encryption_by_default {
```

```terraform
        sse_algorithm = "AES256"
```

```terraform
      }
```

```terraform
    }
```

```terraform
  }
```

```terraform
}
```

[](https://jhooq.com/terraform-secure-sensitive-data/# "Copy Code")[](https://jhooq.com/terraform-secure-sensitive-data/# "Toggle Line Numbers")

terraform

In the above example, we're creating a private **S3 bucket** and enabling server-side encryption using the **AES256** algorithm.

  

### 2.3 Encrypting Sensitive Data in Transit[](https://jhooq.com/terraform-secure-sensitive-data/#23-encrypting-sensitive-data-in-transit)

When it comes to encrypting data in **transit**, **Transport Layer Security (TLS)** is one of the most commonly used protocols.

AWS offers an option to enforce the use of **HTTPS (which incorporates TLS)** in the communication with your **S3 bucket**.

In order to enforce **HTTPS** for an **S3 bucket**, we can use a bucket policy that denies all **non-HTTPS requests**:

```terraform
# Deny all non-https request
```

```terraform
resource "aws_s3_bucket" "bucket" {
```

```terraform
  bucket = "my_tfstate_bucket"
```

```terraform
  acl    = "private"
```

```terraform
}
```

```terraform
resource "aws_s3_bucket_policy" "bucket_policy" {
```

```terraform
  bucket = aws_s3_bucket.bucket.id
```

```terraform
  # Deny non-https request policy
```

```terraform
  policy = jsonencode({
```

```terraform
    Version = "2012-10-17"
```

```terraform
    Statement = [
```

```terraform
      {
```

```terraform
        Sid       = "ForceSSLOnlyAccess"
```

```terraform
        Effect    = "Deny"
```

```terraform
        Principal = "*"
```

```terraform
        Action    = "s3:*"
```

```terraform
        Resource  = "arn:aws:s3:::my_tfstate_bucket/*"
```

```terraform
        Condition = {
```

```terraform
          Bool = {
```

```terraform
            "aws:SecureTransport" = "false"
```

```terraform
          }
```

```terraform
        }
```

```terraform
      },
```

```terraform
    ]
```

```terraform
  })
```

```terraform
}
```

[](https://jhooq.com/terraform-secure-sensitive-data/# "Copy Code")[](https://jhooq.com/terraform-secure-sensitive-data/# "Toggle Line Numbers")

terraform

  

### 2.4 The use of sensitive argument in Terraform to prevent logging of sensitive data[](https://jhooq.com/terraform-secure-sensitive-data/#24-the-use-of-sensitive-argument-in-terraform-to-prevent-logging-of-sensitive-data)

Now, imagine you're a detective (Sherlock Holmes, perhaps?), and your **logs** are like your case notes. They tell you the story of what happened in your system.

But there are certain details - the secret stuff - that you don't want to be written down in your notes for everyone to see. That's exactly where the **sensitive argument** in Terraform comes to the rescue!

Let's see how it works through a practical example:

Consider a scenario where you are creating a new **[AWS RDS](https://aws.amazon.com/rds/)** instance using Terraform. One of the necessary input variables would be your **database password**. We certainly wouldn't want this information being printed in our console output or logs.

```terraform
# make the db_password sensitive
```

```terraform
variable "db_password" {
```

```terraform
  description = "The password for our database."
```

```terraform
  type        = string
```

```terraform
  sensitive   = true
```

```terraform
}
```

```terraform
resource "aws_db_instance" "my_db" {
```

```terraform
  allocated_storage    = 20
```

```terraform
  storage_type         = "gp2"
```

```terraform
  engine               = "mysql"
```

```terraform
  engine_version       = "5.7"
```

```terraform
  instance_class       = "db.t2.micro"
```

```terraform
  name                 = "my_db"
```

```terraform
  username             = "admin"
```

```terraform
  password             = var.db_password
```

```terraform
  parameter_group_name = "default.mysql5.7"
```

```terraform
  skip_final_snapshot  = true
```

```terraform
}
```

[](https://jhooq.com/terraform-secure-sensitive-data/# "Copy Code")[](https://jhooq.com/terraform-secure-sensitive-data/# "Toggle Line Numbers")

terraform

In this example, we've set the **sensitive attribute** of the **db_password** variable to **true**. This ensures that Terraform will redact this value from the logs and console outputs, keeping our secret safe and sound.

However, that's not the end of the story. It's also crucial to understand that Terraform can't completely prevent sensitive data from appearing in logs or command outputs, especially if you are explicitly outputting it.

So, it's essential to use the **sensitive flag** for **output variables** as well, like so:

```terraform
# set sensitive = true for output variable
```

```terraform
output "db_password" {
```

```terraform
   value     = aws_db_instance.my_db.password
```

```terraform
   sensitive = true
```

```terraform
} 
```

[](https://jhooq.com/terraform-secure-sensitive-data/# "Copy Code")[](https://jhooq.com/terraform-secure-sensitive-data/# "Toggle Line Numbers")

terraform

This prevents the password from being exposed in the **terraform apply** output or when you run terraform output.

So, remember, my friends, the **[sensitive argument](https://developer.hashicorp.com/terraform/tutorials/configuration-language/sensitive-variables)** is a really handy tool in our Terraform toolkit. But like any tool, it's up to us to use it wisely.

  

### 2.5 Remote state storage and encryption[](https://jhooq.com/terraform-secure-sensitive-data/#25-remote-state-storage-and-encryption)

Securing your Terraform state files is like putting your gold in a **vault** (no pun intended) - it's a must!

Terraform keeps track of your infrastructure's state in what it calls, well, the state file. By default, this file is stored **locally**, which is not ideal, especially when you're working in a team.

A more secure and collaborative approach is to store the **state file remotely**. For this, we can use a **Terraform backend**.

In [AWS](https://aws.com) land, a popular choice for a remote backend is the **[S3 bucket](https://aws.amazon.com/s3/)**. Here's an example of how to configure S3 as your remote backend:

```terraform
# Remote S3 Bucket to store state file
```

```terraform
terraform {
```

```terraform
   backend "s3" {
```

```terraform
      # S3 bucket hosted on AWS
```

```terraform
      bucket = "my-terraform-state"
```

```terraform
      # Directory inside S3 Bucket
```

```terraform
      key    = "path/to/my/key"
```

```terraform
      region = "us-west-2"
```

```terraform
   }
```

```terraform
} 
```

[](https://jhooq.com/terraform-secure-sensitive-data/# "Copy Code")[](https://jhooq.com/terraform-secure-sensitive-data/# "Toggle Line Numbers")

terraform

In this code snippet, my-terraform-state is the name of the **[S3 bucket](https://aws.amazon.com/s3/)**, **path/to/my/key** is the location of your state file within the bucket, and **us-west-2** is the **AWS** region where your bucket resides.

But, **we're not done yet!** Just storing our state file remotely isn't enough. We should also **encrypt** it at rest. Thankfully, AWS makes this pretty easy for us with **S3 bucket encryption**:

```terraform
# Set he encrypt flag true
```

```terraform
terraform {
```

```terraform
  backend "s3" {
```

```terraform
    bucket  = "my-terraform-state"
```

```terraform
    key     = "path/to/my/key"
```

```terraform
    region  = "us-west-2"
```

```terraform
    encrypt = true
```

```terraform
  }
```

```terraform
} 
```

[](https://jhooq.com/terraform-secure-sensitive-data/# "Copy Code")[](https://jhooq.com/terraform-secure-sensitive-data/# "Toggle Line Numbers")

terraform

See that **encrypt = true** at the end? That's all you need to ensure your state file is encrypted at rest in your **S3 bucket**.

Also, when working in a team, you would want to prevent any conflicts in the state file. This is where state locking comes in. If you enable state locking, only one person can modify the state at a time.

In AWS, we can use [DynamoDB for state locking](https://jhooq.com/terraform-state-file-locking):

```terraform
# State locking using dynamo db
```

```terraform
terraform {
```

```terraform
  backend "s3" {
```

```terraform
    bucket         = "my-terraform-state"
```

```terraform
    key            = "path/to/my/key"
```

```terraform
    region         = "us-west-2"
```

```terraform
    encrypt        = true
```

```terraform
    dynamodb_table = "my-lock-table"
```

```terraform
  }
```

```terraform
} 
```

[](https://jhooq.com/terraform-secure-sensitive-data/# "Copy Code")[](https://jhooq.com/terraform-secure-sensitive-data/# "Toggle Line Numbers")

terraform

In this case, **my-lock-table** is the name of the DynamoDB table that will keep track of the state lock.

So, there you have it, my friends! By storing your state file remotely, encrypting it, and enabling state locking, you're putting that gold safely in a vault.

> **Learn more about -** Terraform state locking using DynamoDB(LockID) and S3 Bucket

  

## 3. Practical Examples of Securing Sensitive Data[](https://jhooq.com/terraform-secure-sensitive-data/#3-practical-examples-of-securing-sensitive-data)

### 3.1 Using environment variables for sensitive data[](https://jhooq.com/terraform-secure-sensitive-data/#31-using-environment-variables-for-sensitive-data)

Using environment variables is like having a secret handshake - you don't write it down, but you and your pals know it by heart.

Similarly, we can use **environment variables** to handle **sensitive data** in Terraform, preventing us from exposing the data directly in our Terraform files.

Terraform can automatically fetch the values for your variables if you've stored them as **environment variables**. You just need to prefix the variable name with **'TF_VAR_'**. Here's how to set this up:

```terraform
export TF_VAR_db_password="MyS3cretP@ssword" 
```

[](https://jhooq.com/terraform-secure-sensitive-data/# "Copy Code")[](https://jhooq.com/terraform-secure-sensitive-data/# "Toggle Line Numbers")

terraform

In this case, **db_password** is our variable name in Terraform, and we are setting the value as **MyS3cretP@ssword**.

Let's assume we have a Terraform configuration to create an AWS RDS instance:

```terraform
# using environment variable to get the password
```

```terraform
variable "db_password" {
```

```terraform
  description = "Password for the database"
```

```terraform
  type        = string
```

```terraform
  sensitive   = true
```

```terraform
}
```

```terraform
resource "aws_db_instance" "default" {
```

```terraform
  allocated_storage    = 10
```

```terraform
  engine               = "mysql"
```

```terraform
  engine_version       = "5.7"
```

```terraform
  instance_class       = "db.t2.micro"
```

```terraform
  name                 = "my_db"
```

```terraform
  username             = "admin"
```

```terraform
  password             = var.db_password
```

```terraform
  parameter_group_name = "default.mysql5.7"
```

```terraform
  skip_final_snapshot  = true
```

```terraform
}
```

[](https://jhooq.com/terraform-secure-sensitive-data/# "Copy Code")[](https://jhooq.com/terraform-secure-sensitive-data/# "Toggle Line Numbers")

terraform

Here, we are using the **db_password** variable as the password for our RDS instance. When we run **terraform apply**, Terraform will automatically fetch the password from our environment variable **TF_VAR_db_password**.

Remember, storing your environment variables securely is also crucial. You might want to consider using a tool like **direnv** or storing them securely in your **CI/CD environment**.

And that's how you manage your secrets with environment variables in Terraform! No paper trails left behind, and your secret handshake remains a secret.

  

### 3.2 Securing sensitive data with AWS Key Management Service (KMS)[](https://jhooq.com/terraform-secure-sensitive-data/#32-securing-sensitive-data-with-aws-key-management-service-kms)

Let's step into the world of **[AWS Key Management Service (KMS)](https://aws.amazon.com/kms/)**!

Picture this - you have a secret message, and you need to ensure that only certain people can read it. What do you do? Well, in the olden days, you might have used a cipher or a secret code. In the AWS universe, we have [AWS Key Management Service (KMS)](https://aws.amazon.com/kms/)!

KMS is a managed service that allows you to create and control **cryptographic keys**, which you can use to **encrypt** and **decrypt** data. It's integrated with other AWS services making it easier to encrypt data you store in these services and control access to the keys that decrypt it.

Let's walk through an example where we are creating an S3 bucket and encrypting its content with a **KMS key**:

```terraform
# Create S3 bucekt but encrypt its content using KMS key
```

```terraform
resource "aws_kms_key" "my_key" {
```

```terraform
  description             = "This is my KMS key for encrypting an S3 bucket"
```

```terraform
  deletion_window_in_days = 10
```

```terraform
}
```

```terraform
resource "aws_s3_bucket" "bucket" {
```

```terraform
  bucket = "my_bucket"
```

```terraform
  acl    = "private"
```

```terraform
  server_side_encryption_configuration {
```

```terraform
    rule {
```

```terraform
      apply_server_side_encryption_by_default {
```

```terraform
        sse_algorithm     = "aws:kms"
```

```terraform
        kms_master_key_id = aws_kms_key.my_key.arn
```

```terraform
      }
```

```terraform
    }
```

```terraform
  }
```

```terraform
}
```

[](https://jhooq.com/terraform-secure-sensitive-data/# "Copy Code")[](https://jhooq.com/terraform-secure-sensitive-data/# "Toggle Line Numbers")

terraform

In this example, we first create a **KMS key** with the **"aws_kms_key"** resource.

Then, when we create our **S3 bucket**, we specify a **server_side_encryption_configuration** block, where we choose **aws:kms** as our **sse_algorithm** and provide the **ARN** of our KMS key as **kms_master_key_id**.

By doing this, any object that's uploaded to our bucket will automatically be encrypted using the KMS key we created. And only those with the necessary permissions to use this KMS key will be able to decrypt the data.

Do note that **KMS keys have a cost associated with them**, and you need to manage permissions for the key separately. So, be careful and vigilant about who can access what!

And that's it, my friends! With **AWS KMS**, your sensitive data gets an extra layer of security.

  

### 3.3 Implementing Vault for secrets management[](https://jhooq.com/terraform-secure-sensitive-data/#33-implementing-vault-for-secrets-management)

Think of [**HashiCorp Vault**](https://www.vaultproject.io/) like your very own digital Swiss bank. It's a centralized service for securely storing and accessing **secrets**, whether they're **API keys**, **passwords**, or **certificates**.

Vault can be particularly useful when working with **Terraform**. You can use it to **dynamically generate secrets**, so you're not hard-coding them into your Terraform scripts. Plus, it has detailed audit logs so you can track who's accessing what.

Here's an example of how to use Vault with Terraform:

First, you need to configure the Vault provider:

```terraform
# Hashicorp vault provider
```

```terraform
provider "vault" {
```

```terraform
  # it is recommended to use environment variables for VAULT_ADDR and VAULT_TOKEN
```

```terraform
  # address = "http://localhost:8200"
```

```terraform
  # token   = "s.myVaultToken123456"
```

```terraform
} 
```

[](https://jhooq.com/terraform-secure-sensitive-data/# "Copy Code")[](https://jhooq.com/terraform-secure-sensitive-data/# "Toggle Line Numbers")

terraform

For the Vault address and token, it's a best practice to use environment variables (`VAULT_ADDR` and `VAULT_TOKEN` respectively). This way, you're not hardcoding these potentially sensitive details into your scripts.

Now, let's assume you have a secret stored in Vault at path `secret/my_secrets/db_password`. Here's how you would retrieve it with Terraform:

```terraform
# set the password
```

```terraform
data "vault_generic_secret" "db_password" {
```

```terraform
  path = "secret/my_secrets/db_password"
```

```terraform
}
```

```terraform
output "db_password" {
```

```terraform
  value     = data.vault_generic_secret.db_password.data["value"]
```

```terraform
  sensitive = true
```

```terraform
} 
```

[](https://jhooq.com/terraform-secure-sensitive-data/# "Copy Code")[](https://jhooq.com/terraform-secure-sensitive-data/# "Toggle Line Numbers")

terraform

In this example, the **vault_generic_secret** data source is used to fetch the secret from Vault. The value of the secret can then be accessed with **data.vault_generic_secret.db_password.data["value"]**.

We're also using the sensitive attribute for our output to ensure that the password won't be shown in the **terraform apply** output.

But remember, the Swiss bank isn't invincible, and neither is Vault. You'll need to secure your Vault instance - think about encryption, access policies, and regular auditing. Use it wisely, and it can be a powerful tool in your Terraform toolkit.

> **Here are more concrete lab sessions and playlist on how to use Hashicorp Vault**
> 
> 1. [HashiCorp Vault Installation - Part 1](https://youtu.be/-sU0O82fdZs)
> 2. [HashiCorp Vault Start and Stop in Development mode - Part 2](https://youtu.be/DvwJEuUS540)
> 3. [HashiCorp Vault Read Write and Delete secrets - Part 3](https://youtu.be/SvyIkMDsdUY)
> 4. [HashiCorp Vault Secret Engine and Secret Engine path - Part 4](https://youtu.be/V-YaXal0t4k)
> 5. [HashiCorp Vault Dynamic Secrets generation - Part 5](https://youtu.be/mASG1rIHa5U)
> 6. [HashiCorp Vault Token Authentication & GitHub Authentication - Part 6 | HashiCorp Vault tutorials](https://youtu.be/LX2YrBbGOGQ)
> 7. [HashiCorp Vault Policy - Part 7 | HashiCorp Vault tutorials](https://youtu.be/Opdq8YPRLBQ)
> 8. [HashiCorp Vault Deploy Vault, HTTP API & UI - Part 8](https://youtu.be/OyL4NpM_BPA)

  

## 4. Regular Auditing and Rotating Secrets[](https://jhooq.com/terraform-secure-sensitive-data/#4-regular-auditing-and-rotating-secrets)

### 4.1 The importance of auditing[](https://jhooq.com/terraform-secure-sensitive-data/#41-the-importance-of-auditing)

Let's step into the shoes of our inner security guard and explore the world of auditing and secret rotation.

Imagine you're a detective on a **TV show**, and you've got a case full of evidence. To solve the mystery, you need to look at **each piece of evidence** carefully, figure out what it tells you. In the world of cloud infrastructure, this is akin to auditing.

Regular auditing is like regularly checking that evidence room, making sure nothing's out of place, and everything's accounted for. You're **reviewing logs**, **looking for any suspicious activity**, and verifying **who accessed what and when**.

**In Terraform, you might want to audit your state file** - The state file is like a treasure map - it tells you what resources you have and their configurations. It can contain **sensitive information**, so it's crucial to know who's accessing it.

For example, if you're using **S3** as your **remote backend**, you can enable **access logging** and use **CloudTrail** to track access to your state file. This will give you valuable insights, like who accessed the state file, when they accessed it, and what actions they performed.

  

### 4.2 Regular rotation of secrets[](https://jhooq.com/terraform-secure-sensitive-data/#42-regular-rotation-of-secrets)

Just like a master of disguise changes their appearance regularly to avoid detection, we need to **rotate our secrets regularly**. This could be your **database passwords, API keys, or tokens** - anything that provides access to your resources.

Rotating secrets reduces the risk of them being misused. **If a secret is exposed but you rotate it soon after, the exposed secret becomes useless.**

**Terraform doesn't directly help with secret rotation**, but [Vault](https://www.vaultproject.io/), a tool we discussed earlier, shines here. Vault has a feature called dynamic secrets. It can generate secrets on-the-fly and automatically revoke them after a certain period. You can use this feature to regularly rotate your secrets.

  

## 5. Conclusion[](https://jhooq.com/terraform-secure-sensitive-data/#5-conclusion)

First, we embarked on the noble path of **"Understanding sensitive data in Terraform"**, where we grasped why it's critical to protect sensitive data and the risks if we don't. Our quest then led us to decipher the enigma of **"Encrypting sensitive data in transit and at rest"**, unveiling the twin shields of **SSL/TLS** and **AWS Key Management Service (KMS)**.

Next, our journey took a slightly esoteric turn as we delved into the cosmos of Terraform to discover the magic of **"sensitive" argument**, an incantation that prevents our secrets from appearing in Terraform's output.

Our path then led us to the heart of the Terraform kingdom - **the "State File"**. We learnt how to secure it by using **remote backends** and **encryption**, storing it far away from prying eyes, and protecting our precious map to the treasure.

We then navigated the labyrinth of **environment variables**, turning them into our allies to handle **sensitive data securely and elegantly**.

Barely pausing for breath, we ventured into the world of **AWS Key Management Service (KMS)**, learning to wield it as a sword that encrypts our data with unbreakable cryptographic keys.

Our next destination was the [**HashiCorp Vault**](https://www.vaultproject.io/), an impregnable fortress that safely stores our secrets, only releasing them to those deemed worthy.

Our journey was nearing its end, but not before we unravelled the mystery of encrypting **[Terraform state files](https://jhooq.com/terraform-manage-state)**. We learnt how to make our treasure map indecipherable to anyone but us.

Finally, we took on the role of vigilant watchmen, keeping a sharp eye on our infrastructure through regular auditing and changing our secret codes frequently by rotating secrets.

## Posts in this series

- [Securing Sensitive Data in Terraform](https://jhooq.com/terraform-secure-sensitive-data/ "Securing Sensitive Data in Terraform")
- [Boost Your AWS Security with Terraform : A Step-by-Step Guide](https://jhooq.com/security-practices-aws-terraform/ "Boost Your AWS Security with Terraform : A Step-by-Step Guide")
- [How to Load Input Data from a File in Terraform?](https://jhooq.com/terraform-load-data-from-input-file/ "How to Load Input Data from a File in Terraform?")
- [Can Terraform be used to provision on-premises infrastructure?](https://jhooq.com/terraform-provision-on-premise-infra/ "Can Terraform be used to provision on-premises infrastructure?")
- [Fixing the Terraform Error creating IAM Role. MalformedPolicyDocument Has prohibited field Resource](https://jhooq.com/terraform-malformed-policy-document/ "Fixing the Terraform Error creating IAM Role. MalformedPolicyDocument Has prohibited field Resource")
- [In terraform how to handle null value with default value?](https://jhooq.com/terraform-replace-null-value-with-default-value/ "In terraform how to handle null value with default value?")
- [Terraform use module output variables as inputs for another module?](https://jhooq.com/terraform-ouput-module-input-another-module/ "Terraform use module output variables as inputs for another module?")
- [How to Reference a Resource Created by a Terraform Module?](https://jhooq.com/terraform-reference-module-output/ "How to Reference a Resource Created by a Terraform Module?")
- [Understanding Terraform Escape Sequences](https://jhooq.com/terraform-escape-sequences/ "Understanding Terraform Escape Sequences")
- [How to fix private-dns-enabled cannot be set because there is already a conflicting DNS domain?](https://jhooq.com/terraform-error-private-dbs-enabled/ "How to fix private-dns-enabled cannot be set because there is already a conflicting DNS domain?")
- [Use Terraform to manage AWS IAM Policies, Roles and Users](https://jhooq.com/terraform-manage-user-roles-policy/ "Use Terraform to manage AWS IAM Policies, Roles and Users")
- [How to split Your Terraform main.tf File into Multiple Files](https://jhooq.com/terraform-split-main-tf/ "How to split Your Terraform main.tf File into Multiple Files")
- [How to use Terraform variable within variable](https://jhooq.com/terraform-var-within-var/ "How to use Terraform variable within variable")
- [Mastering the Terraform Lookup Function for Dynamic Keys](https://jhooq.com/terraform-lookup-function/ "Mastering the Terraform Lookup Function for Dynamic Keys")
- [Copy files to EC2 and S3 bucket using Terraform](https://jhooq.com/terraform-copy-file-ec2-s3/ "Copy files to EC2 and S3 bucket using Terraform")
- [Troubleshooting Error creating EC2 Subnet InvalidSubnet Range The CIDR is Invalid](https://jhooq.com/terraform-invalid-subnet/ "Troubleshooting Error creating EC2 Subnet InvalidSubnet Range The CIDR is Invalid")
- [Troubleshooting InvalidParameter Security group and subnet belong to different networks](https://jhooq.com/terraform-security-group-with-different-subnet/ "Troubleshooting InvalidParameter Security group and subnet belong to different networks")
- [Managing strings in Terraform: A comprehensive guide](https://jhooq.com/terraform-string-handling/ "Managing strings in Terraform: A comprehensive guide")
- [How to use terraform depends_on meta argument?](https://jhooq.com/terraform-depends-on/ "How to use terraform depends_on meta argument?")
- [What is user_data in Terraform?](https://jhooq.com/terraform-user-data/ "What is user_data in Terraform?")
- [Why you should not store terraform state file(.tfstate) inside Git Repository?](https://jhooq.com/terraform-do-not-store-tfstate-in-git/ "Why you should not store terraform state file(.tfstate) inside Git Repository?")
- [How to import existing resource using terraform import comand?](https://jhooq.com/terraform-import-resource/ "How to import existing resource using terraform import comand?")
- [Terraform - A detailed guide on setting up ALB(Application Load Balancer) and SSL?](https://jhooq.com/terraform-guide-alb-ssl/ "Terraform - A detailed guide on setting up ALB(Application Load Balancer) and SSL?")
- [Testing Infrastructure as Code with Terraform?](https://jhooq.com/terraform-testing-infra-as-code/ "Testing Infrastructure as Code with Terraform?")
- [How to remove a resource from Terraform state?](https://jhooq.com/terraform-remove-resource-from-state/ "How to remove a resource from Terraform state?")
- [What is Terraform null Resource?](https://jhooq.com/terraform-null-resource/ "What is Terraform null Resource?")
- [In terraform how to skip creation of resource if the resource already exist?](https://jhooq.com/terraform-check-if-resource-exist/ "In terraform how to skip creation of resource if the resource already exist?")
- [How to setup Virtual machine on Google Cloud Platform](https://jhooq.com/how-to-setup-virtual-machine-on-google-cloud-platform/ "How to setup Virtual machine on Google Cloud Platform")
- [How to use Terraform locals?](https://jhooq.com/how-to-use-terraform-locals/ "How to use Terraform locals?")
- [Terraform Guide - Docker Containers & AWS ECR(elastic container registry)?](https://jhooq.com/terraform-docker-container-guide/ "Terraform Guide - Docker Containers & AWS ECR(elastic container registry)?")
- [How to generate SSH key in Terraform using tls_private_key?](https://jhooq.com/terraform-generate-ssh-key/ "How to generate SSH key in Terraform using tls_private_key?")
- [How to fix-Terraform Error acquiring the state lock ConditionalCheckFiledException?](https://jhooq.com/terraform-conidtional-check-failed/ "How to fix-Terraform Error acquiring the state lock ConditionalCheckFiledException?")
- [Terraform Template - A complete guide?](https://jhooq.com/terraform-template/ "Terraform Template - A complete guide?")
- [How to use Terragrunt?](https://jhooq.com/terragrunt-guide/ "How to use Terragrunt?")
- [Terraform and AWS Multi account Setup?](https://jhooq.com/terraform-aws-multi-account/ "Terraform and AWS Multi account Setup?")
- [Terraform and AWS credentials handling?](https://jhooq.com/terraform-aws-credentials-handling/ "Terraform and AWS credentials handling?")
- [How to fix-error configuring S3 Backend no valid credential sources for S3 Backend found?](https://jhooq.com/terraform-error-config-s3/ "How to fix-error configuring S3 Backend no valid credential sources for S3 Backend found?")
- [Terraform state locking using DynamoDB (aws_dynamodb_table)?](https://jhooq.com/terraform-state-file-locking/ "Terraform state locking using DynamoDB (aws_dynamodb_table)?")
- [Managing Terraform states?](https://jhooq.com/terraform-manage-state/ "Managing Terraform states?")
- [Securing AWS secrets using HashiCorp Vault with Terraform?](https://jhooq.com/hashi-vault-aws-secret-terraform/ "Securing AWS secrets using HashiCorp Vault with Terraform?")
- [How to use Workspaces in Terraform?](https://jhooq.com/terraform-workspaces/ "How to use Workspaces in Terraform?")
- [How to run specific terraform resource, module, target?](https://jhooq.com/terraform-run-specific-resource/ "How to run specific terraform resource, module, target?")
- [How Terraform modules works?](https://jhooq.com/terraform-module/ "How Terraform modules works?")
- [Secure AWS EC2s & GCP VMs with Terraform SSH Keys!](https://jhooq.com/terraform-ssh-into-aws-ec2/ "Secure AWS EC2s & GCP VMs with Terraform SSH Keys!")
- [What is terraform provisioner?](https://jhooq.com/terraform-provisioner/ "What is terraform provisioner?")
- [Is terraform destroy needed before terraform apply?](https://jhooq.com/terraform-destroy-before-apply/ "Is terraform destroy needed before terraform apply?")
- [How to fix terraform error Your query returned no results. Please change your search criteria and try again?](https://jhooq.com/terraform-your-query-returned-no-results/ "How to fix terraform error Your query returned no results. Please change your search criteria and try again?")
- [How to use Terraform Data sources?](https://jhooq.com/terraform-data-sources/ "How to use Terraform Data sources?")
- [How to use Terraform resource meta arguments?](https://jhooq.com/terraform-resource-meta-argument/ "How to use Terraform resource meta arguments?")
- [How to use Terraform Dynamic blocks?](https://jhooq.com/terraform-dynamic-block/ "How to use Terraform Dynamic blocks?")
- [Terraform - How to nuke AWS resources and save additional AWS infrastructure cost?](https://jhooq.com/terraform-nuke/ "Terraform - How to nuke AWS resources and save additional AWS infrastructure cost?")
- [Understanding terraform count, for_each and for loop?](https://jhooq.com/terraform-for-and-for-each-loop/ "Understanding terraform count, for_each and for loop?")
- [How to use Terraform output values?](https://jhooq.com/how-to-use-terraform-output-values/ "How to use Terraform output values?")
- [How to fix error configuring Terraform AWS Provider error validating provider credentials error calling sts GetCallerIdentity SignatureDoesNotMatch?](https://jhooq.com/error-configuring-terraform-aws-provider/ "How to fix error configuring Terraform AWS Provider error validating provider credentials error calling sts GetCallerIdentity SignatureDoesNotMatch?")
- [How to fix Invalid function argument on line in provider credentials file google Invalid value for path parameter no file exists](https://jhooq.com/gcp-invalid-function-argument-credential-file/ "How to fix Invalid function argument on line in provider credentials file google Invalid value for path parameter no file exists")
- [How to fix error value for undeclared variable a variable named was assigned on the command line?](https://jhooq.com/value-for-undeclared-variable/ "How to fix error value for undeclared variable a variable named was assigned on the command line?")
- [What is variable.tf and terraform.tfvars?](https://jhooq.com/terraform-variable-and-tfvars-file/ "What is variable.tf and terraform.tfvars?")
- [How to use Terraform Variables - Locals,Input,Output](https://jhooq.com/terraform-input-variables/ "How to use Terraform Variables - Locals,Input,Output")
- [Terraform create EC2 Instance on AWS](https://jhooq.com/terraform-ec2-instance-setup/ "Terraform create EC2 Instance on AWS")
- [How to fix Error creating service account googleapi Error 403 Identity and Access Management (IAM) API has not been used in project before or it is disabled](https://jhooq.com/fix-error-create-service-account-google-api/ "How to fix Error creating service account googleapi Error 403 Identity and Access Management (IAM) API has not been used in project before or it is disabled")
- [Install terraform on Ubuntu 20.04, CentOS 8, MacOS, Windows 10, Fedora 33, Red hat 8 and Solaris 11](https://jhooq.com/install-terrafrom/ "Install terraform on Ubuntu 20.04, CentOS 8, MacOS, Windows 10, Fedora 33, Red hat 8 and Solaris 11")