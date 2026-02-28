🚀 GitHub to S3 Sync with AWS CodePipeline
This project provides a professional-grade automation setup to connect your GitHub repository to an AWS S3 bucket. Every time you push code to your selected branch, AWS CodePipeline will automatically deploy the latest version to your static website.

📋 Prerequisites
An AWS Account with administrative access.

A GitHub Repository containing your website files (e.g., index.html).

🛠 Phase 1: Prepare the S3 Destination
This bucket will host your static website.

1. Create the Bucket
Go to the S3 Console.

Create a bucket with a unique name (e.g., my-github-website-2026).

2. Configure Permissions
Public Access: Uncheck Block all public access and acknowledge the warning.

Static Hosting: * Navigate to the Properties tab.

Scroll to the bottom and Enable "Static website hosting."

Set index.html as the Index document.

3. Apply Bucket Policy
In the Permissions tab, add the following JSON to allow public read access. Replace YOUR-BUCKET-NAME with your actual bucket name:

JSON

{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Sid": "PublicReadGetObject",
            "Effect": "Allow",
            "Principal": "*",
            "Action": "s3:GetObject",
            "Resource": "arn:aws:s3:::YOUR-BUCKET-NAME/*"
        }
    ]
}
🔗 Phase 2: Create the Pipeline (GitHub → S3)
1. Pipeline Basics
Go to CodePipeline > Create pipeline.

Name: GitHub-to-S3-Sync.

Service Role: Select New service role. AWS will automatically generate the required IAM permissions.

2. Source Stage (GitHub)
Provider: GitHub (Version 2).

Connection: Click Connect to GitHub.

Provide a connection name.

Click Connect to GitHub in the popup and authorize AWS.

Click Install a new app to select your specific repository.

Setup: Select your Repository and Branch (usually main).

3. Build Stage
Click Skip build stage.

Note: You only need this stage if you are using frameworks like React, Vue, or Hugo that require a compilation step.

4. Deploy Stage
Provider: Amazon S3.

Bucket: Choose the website bucket you created in Phase 1.

S3 Deployment Options: Check the box Extract file before deploying.

Why? This ensures your files are placed in the bucket individually instead of as a single .zip archive.

✅ Final Steps
Review and click Create pipeline.

Wait for the initial execution to complete.

Visit your S3 Bucket Website Endpoint (found under the Properties tab) to see your live site!
