# Challenge 1: OpenTofu Remote State Security with AWS KMS Encryption

In this challenge, you will configure OpenTofu (previously known as Terraform) to securely store its remote state in an Amazon S3 bucket, using AWS Key Management Service (KMS) for encryption. This will ensure that your infrastructure state is protected with strong encryption, enhancing the overall security of your infrastructure management process.

## 1. Create an S3 Bucket for Remote State
- Create an S3 bucket to store the OpenTofu remote state files.
- Enable server-side encryption with AWS KMS for the bucket.

## 2. Set Up an AWS KMS Key
- Create a KMS key to be used for encrypting the S3 bucket.
- Configure key policies to allow appropriate access.

## 3. Configure OpenTofu to Use the S3 Backend
- Configure OpenTofu to use the S3 bucket as the backend for remote state storage.
- Ensure that the state is encrypted using the AWS KMS key.

## 4. Demonstrate Secure State Storage
- Deploy a simple infrastructure using OpenTofu to verify the configuration.
- Show that the state file is stored in the S3 bucket and is encrypted with KMS.
