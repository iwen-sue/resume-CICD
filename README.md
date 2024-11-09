# **Jekyll CI/CD Pipeline Using AWS CodePipeline**

This documentation outlines the architecture, setup steps, and maintenance practices for a CI/CD pipeline that automates the deployment of a Jekyll site hosted in an **Amazon S3** bucket, using **AWS CodePipeline** and **GitHub** as the source repository.

---

## **Pipeline Architecture**

### **Overview**

The pipeline automates:

1. **Source Management**: Fetching code from a GitHub repository.
2. **Build Process**: Running Jekyll to generate static files.
3. **Deployment**: Uploading the generated static files to an S3 bucket under a specified folder.

### **Architecture Diagram**

```python
+-----------------+             +------------------+             +---------------------+
|   GitHub Repo   |             |  AWS CodeBuild   |             |        S3           |
|  (Source Code)  |    --->     |  (Generate Site) |    --->     |  (Host /resume)     |
+-----------------+             +------------------+             +---------------------+
       Webhook                         Buildspec                          Static Site
```

### **AWS Services Used**

1. **CodePipeline**: Orchestrates the pipeline workflow.
2. **CodeBuild**: Builds the Jekyll site and organizes files.
3. **S3**: Hosts the static website and serves it publicly.

---

## **Pipeline Setup**

### **Step 1: Prepare AWS Resources**

### **1. Create an S3 Bucket**

- Log in to the AWS Management Console.
- Navigate to **S3**.
- Create a bucket (e.g., `my-jekyll-site-bucket`).
- Enable **Static Website Hosting** under the bucket's **Properties**.
- Set the **Index Document** to `resume/index.html`.

### **2. Set Up S3 Bucket Permissions**

Configure the bucket policy to allow public read access for objects:

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "PublicReadGetObject",
      "Effect": "Allow",
      "Principal": "*",
      "Action": "s3:GetObject",
      "Resource": "arn:aws:s3:::my-jekyll-site-bucket/*"
    }
  ]
}
```

---

### **3. Create an IAM Role for CodePipeline**

Create a service role for **CodePipeline**:

- Attach the managed policy `AWSCodePipelineFullAccess`.

### **4. Create an IAM Role for CodeBuild**

Create a service role for **CodeBuild**:

- Attach the managed policies:
    - `AWSCodeBuildDeveloperAccess`
    - `AmazonS3FullAccess`

---

### **Step 2: Set Up the Pipeline**

### **1. Create a CodePipeline**

- Navigate to **CodePipeline** in the AWS Management Console.
- Click **Create Pipeline** and provide a name (e.g., `JekyllPipeline`).

### **Source Stage (GitHub)**

- **Source Provider**: Select **GitHub Version 2**.
- **Connect to GitHub**: Authenticate and select the repository and branch (e.g., `main`).
- Set up a **webhook** for automatic triggers.

### **Build Stage (CodeBuild)**

- **Build Provider**: Select **AWS CodeBuild**.
- **Create a CodeBuild Project**:
    - Environment:
        - Runtime: Ubuntu with Ruby support.
        - Image: `aws/codebuild/standard:5.0` (or latest).
    - **Buildspec File**: Select "Use a buildspec file."

### **Deploy Stage (S3)**

- **Deploy Provider**: Select **Amazon S3**.
- Specify the target bucket (e.g., `my-jekyll-site-bucket`) and folder (`resume`).

---

### **Step 3: Configure Buildspec File**

Add the following `buildspec.yml` file to your GitHub repository root:

```yaml
version: 0.2

phases:
  install:
    runtime-versions:
      ruby: 2.7
    commands:
      - gem install bundler
      - bundle install
  build:
    commands:
      - bundle exec jekyll build
      - mkdir -p build_output/resume
      - mv _site/* build_output/resume
artifacts:
  files:
    - '**/*'
  base-directory: 'build_output'
```

---

### **Step 4: Deploy and Test**

1. Push changes to the GitHub repository.
2. Monitor the pipeline in the AWS Console.
3. Access the site:
    
    ```bash
    http://your-bucket-name.s3-website-region.amazonaws.com/resume/resume.html
    ```
    

---

## **Pipeline Maintenance**

### **1. Monitor Pipeline Execution**

- Use the **CodePipeline Console** to monitor each stage.
- Check logs for failed builds in the **CodeBuild Console**.

### **2. Update the Pipeline**

- **Source Changes**:
    - Modify the GitHub branch or repository in the pipeline source stage.
- **Build Changes**:
    - Update the `buildspec.yml` file to include new build steps.
- **Deployment Changes**:
    - Adjust S3 bucket or folder configurations in the deployment stage.

### **3. Manage Dependencies**

- Periodically update Ruby gems in your project by running:
    
    ```bash
    bundle update
    ```
    

### **4. Performance Optimization**

- Enable **CloudFront** for faster content delivery.
- Add caching policies for S3 objects.

### **5. Cost Management**

- Monitor usage costs using the **AWS Cost Explorer**.
- Enable **S3 Object Expiration** to clean up old files.

### **6. Security Best Practices**

- Regularly review and limit IAM roles’ permissions.
- Enable **S3 Server Access Logging** for audit purposes.

---

## **Common Issues and Troubleshooting**

| **Issue** | **Cause** | **Solution** |
| --- | --- | --- |
| Build fails in CodeBuild | Missing dependencies or incorrect commands | Check the logs in CodeBuild and verify the `buildspec.yml`. |
| Deployment to S3 fails | Insufficient permissions for the S3 bucket | Ensure CodeBuild and CodePipeline roles have `AmazonS3FullAccess`. |
| Site not accessible publicly | S3 bucket permissions are misconfigured | Update the bucket policy to allow public `s3:GetObject` access. |
| Changes not reflecting | Old files cached by CloudFront | Invalidate the CloudFront cache or use a versioned URL for static assets. |

---

## Current Completions

1. Hosted Website
2. Version Control System
3. CI/CD Pipeline Setup
4. Documentation

## **Future Enhancements**

1. **Add Testing**:
    - Integrate testing tools (e.g., HTML validation) into the pipeline.
2. **CloudFront with SSL**:
    - Use Amazon CloudFront for HTTPS and faster delivery.
3. **Multi-Environment Support**:
    - Set up separate pipelines for staging and production.
4. **Automate Pipeline Updates**:
    - Use AWS CloudFormation to define and manage the pipeline as code.
