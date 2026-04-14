# 🚀 GitHub-to-S3 Automated Sync

Automate the deployment of static websites by connecting GitHub to AWS CodePipeline. Every `git push` triggers an automated deployment to an S3 bucket configured for static hosting — no manual uploads, no downtime.

---

## 🏗️ Architecture Overview

```
┌─────────────────────────────────────────────────────────────────┐
│                        Developer Machine                        │
│                                                                 │
│   index.html / style.css / assets  →  git push origin main     │
└──────────────────────────┬──────────────────────────────────────┘
                           │
                           ▼
┌─────────────────────────────────────────────────────────────────┐
│                         GitHub Repo                             │
│                      (main branch)                              │
└──────────────────────────┬──────────────────────────────────────┘
                           │  Change Detected
                           ▼
┌─────────────────────────────────────────────────────────────────┐
│                  AWS CodeStar Connection                         │
│               (GitHub ↔ AWS Authorization)                      │
└──────────────────────────┬──────────────────────────────────────┘
                           │  Triggers Pipeline
                           ▼
┌─────────────────────────────────────────────────────────────────┐
│                    AWS CodePipeline                             │
│                                                                 │
│   ┌─────────────┐        ┌──────────────┐                      │
│   │   SOURCE    │   →    │    DEPLOY    │   (Build skipped)    │
│   │  (GitHub)   │        │   (S3)       │                      │
│   └─────────────┘        └──────┬───────┘                      │
└──────────────────────────────────┼──────────────────────────────┘
                                   │  Extract & Deploy
                                   ▼
┌─────────────────────────────────────────────────────────────────┐
│                        AWS S3 Bucket                            │
│                  (Static Website Hosting)                       │
│                                                                 │
│   index.html   style.css   /assets/                            │
└──────────────────────────┬──────────────────────────────────────┘
                           │
                           ▼
              🌐 Live Website URL
     http://<bucket-name>.s3-website-<region>.amazonaws.com
```

---

## 📋 Prerequisites

| Requirement | Details |
|---|---|
| **AWS Account** | With administrative privileges |
| **GitHub Repository** | Containing your web assets (`index.html`, `style.css`, etc.) |

---

## 🛠️ Phase 1: Prepare the S3 Destination

### Step 1 – Create the Bucket

1. Open the **S3 Console**
2. Click **Create bucket**
3. Enter a unique bucket name (e.g., `my-github-website-2026`)
4. Choose your preferred AWS region

---

### Step 2 – Configure Settings

#### Public Access
- Uncheck **Block all public access**
- Acknowledge the warning prompt

#### Static Website Hosting
Navigate to **Properties → Static website hosting** and configure:

```
Enable:         ✅ Enable
Index document: index.html
```

---

### Step 3 – Apply Bucket Policy

Navigate to **Permissions → Bucket Policy** and paste the following JSON.
Replace `YOUR-BUCKET-NAME` with your actual bucket name:

```json
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
```

---

## 🔗 Phase 2: Create the CodePipeline

### Step 1 – Pipeline Settings

| Field | Value |
|---|---|
| Pipeline Name | `GitHub-to-S3-Sync` |
| Service Role | New service role |

---

### Step 2 – Source Stage (GitHub)

| Field | Value |
|---|---|
| Provider | GitHub (Version 2) |
| Connection | Click **Connect to GitHub** → authorize AWS |
| Repository | Select your target repository |
| Branch | `main` |

---

### Step 3 – Build Stage

> Click **Skip build stage**

> **Note:** A build stage is only required for frameworks that need compilation (React, Vue, Angular, etc.). For plain HTML/CSS/JS static sites, skip this step.

---

### Step 4 – Deploy Stage

| Field | Value |
|---|---|
| Provider | Amazon S3 |
| Bucket | Select your website bucket |
| Extract file before deploying | ✅ **MUST be checked** |

> ⚠️ If you forget to check **Extract file before deploying**, a `.zip` file will be uploaded to your bucket instead of your actual website files.

---

## ✅ Verify Deployment

Once the pipeline shows a **green Succeeded** status:

1. Go to your **S3 bucket → Properties tab**
2. Scroll to **Static website hosting**
3. Copy the **Bucket website endpoint** URL

```
http://<bucket-name>.s3-website-<region>.amazonaws.com
```

Your site is live! Every future `git push` to `main` will automatically trigger a redeployment.

---

## ⚠️ Troubleshooting

| Problem | Cause | Fix |
|---|---|---|
| **403 Forbidden** | Bucket policy missing or Public Access still blocked | Re-save the bucket policy; ensure **Block all public access** is OFF |
| **Files not updating** | Pipeline source step failed | Open CodePipeline Console → check latest **Source** stage status |
| **Zip file in S3 bucket** | Deploy option not set | Edit the Deploy stage → check **Extract file before deploying** |
| **Pipeline not triggering** | CodeStar Connection not authorized | Reconnect GitHub via **Settings → Connections** in the Developer Tools console |

---

## 📂 Recommended Repository Structure

```
your-repo/
├── index.html        ← Entry point (required)
├── style.css
├── script.js
└── assets/
    ├── images/
    └── fonts/
```

---

## 🔄 Deployment Flow Summary

```
git push  →  CodeStar detects change  →  CodePipeline triggers
          →  Source fetched from GitHub
          →  Files extracted & synced to S3
          →  Website updated instantly  🎉
```
