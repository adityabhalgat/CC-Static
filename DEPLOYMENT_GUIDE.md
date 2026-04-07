# AWS Deployment Guide for Static Website

This guide explains how to deploy this static website on AWS in detail.

## Project Files

Make sure your project folder contains:

- `index.html`
- `styles.css`
- `script.js`

## Option A (Recommended): Deploy with Amazon S3 Static Website Hosting

This is the easiest and most common approach for assignments.

## 1. Create an AWS Account

1. Go to https://aws.amazon.com/ and sign in.
2. Open AWS Management Console.
3. Search for **S3** in the search bar and open it.

## 2. Create an S3 Bucket

1. Click **Create bucket**.
2. Bucket name must be globally unique, for example:
   - `urban-pulse-static-site-2026-yourname`
3. Choose a nearby AWS Region.
4. In **Block Public Access settings**, uncheck:
   - `Block all public access`
5. Confirm warning checkbox.
6. Click **Create bucket**.

## 3. Upload Website Files

1. Open your new bucket.
2. Click **Upload**.
3. Upload these files from your local project:
   - `index.html`
   - `styles.css`
   - `script.js`
4. Click **Upload**.

## 4. Enable Static Website Hosting

1. In the bucket, go to **Properties** tab.
2. Scroll to **Static website hosting**.
3. Click **Edit**.
4. Enable static website hosting.
5. Hosting type: `Host a static website`.
6. Index document: `index.html`.
7. Error document: `index.html` (or create a separate `error.html` if required).
8. Save changes.

AWS will generate a **Bucket website endpoint URL**. Keep this URL.

## 5. Add Public Read Bucket Policy

1. Go to **Permissions** tab in your bucket.
2. In **Bucket policy**, click **Edit**.
3. Paste this policy (replace `YOUR_BUCKET_NAME`):

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "PublicReadGetObject",
      "Effect": "Allow",
      "Principal": "*",
      "Action": "s3:GetObject",
      "Resource": "arn:aws:s3:::YOUR_BUCKET_NAME/*"
    }
  ]
}
```

4. Save policy.

## 6. Test Your Website

1. Open the **Bucket website endpoint URL** in browser.
2. Confirm:
   - Home page loads
   - CSS is applied
   - Mobile menu works

If CSS or JS is not loading, verify file names and paths are exact.

## 7. Updating the Website Later

Any time you change code:

1. Re-upload changed files to the same S3 bucket.
2. Refresh browser (hard refresh: `Cmd + Shift + R` on macOS).

---

## Option B: S3 + CloudFront + Custom Domain + HTTPS (Production Style)

Use this for better performance and HTTPS.

## 1. Keep S3 Setup from Option A

Do all steps from Option A first.

## 2. Create CloudFront Distribution

1. Open **CloudFront** in AWS Console.
2. Click **Create distribution**.
3. Origin domain: choose your S3 website endpoint or S3 bucket endpoint depending on setup.
4. Viewer protocol policy: `Redirect HTTP to HTTPS`.
5. Default root object: `index.html`.
6. Create distribution.

## 3. Attach Custom Domain (Optional)

1. Buy/register domain in **Route 53** or external registrar.
2. Request SSL certificate in **AWS Certificate Manager (ACM)** in `us-east-1`.
3. Add domain to CloudFront Alternate Domain Names (CNAMEs).
4. Attach ACM certificate.
5. Create DNS record in Route 53 pointing domain to CloudFront distribution.

## 4. Cache Invalidation After Updates

When updating files, CloudFront may serve cached content. Invalidate cache:

1. Open CloudFront distribution.
2. Go to **Invalidations**.
3. Create invalidation path:
   - `/*`

---

## Option C: Deploy via AWS Amplify (Git-based, easiest CI/CD)

Use this if your code is in GitHub.

## 1. Push Code to GitHub

Initialize repo and push your static site files.

## 2. Create Amplify App

1. Open **AWS Amplify** console.
2. Click **New app** > **Host web app**.
3. Connect GitHub and choose your repository + branch.
4. Build settings:

```yaml
version: 1
frontend:
  phases:
    build:
      commands: []
  artifacts:
    baseDirectory: /
    files:
      - '**/*'
  cache:
    paths: []
```

5. Save and deploy.

Amplify gives a live URL and automatically redeploys on every git push.

---

## AWS CLI Method (Optional, Fast Repeat Deployments)

If you installed AWS CLI and configured credentials:

## 1. Configure AWS CLI

```bash
aws configure
```

Enter:

- AWS Access Key ID
- AWS Secret Access Key
- Region (for example `ap-south-1`)
- Output format (`json`)

## 2. Sync Local Files to Bucket

Run inside your project folder:

```bash
aws s3 sync . s3://YOUR_BUCKET_NAME --delete --exclude ".git/*" --exclude "*.md"
```

This uploads changes and removes deleted files from bucket.

---

## Submission Checklist (for Assignment)

- Static website built and running locally
- Deployed URL from AWS is working
- CSS and JS loaded correctly
- Mobile responsive check done
- Deployment steps documented (this file)
- Screenshots captured for:
  - S3 bucket settings
  - Static hosting enabled
  - Bucket policy
  - Live website URL

---

## Common Errors and Fixes

1. `403 Forbidden`
- Cause: public access not configured.
- Fix: disable block public access and apply bucket policy.

2. `404 Not Found`
- Cause: missing `index.html` or wrong file name case.
- Fix: ensure exact file name `index.html`.

3. CSS/JS not loading
- Cause: wrong path or not uploaded.
- Fix: keep files in same bucket root and use:
  - `<link rel="stylesheet" href="styles.css">`
  - `<script src="script.js"></script>`

4. Old content still visible
- Cause: browser/CloudFront cache.
- Fix: hard refresh or CloudFront invalidation `/*`.

---

## Final Note

For assignment grading, Option A (S3 static hosting) is usually enough. If your instructor asks for HTTPS and custom domain, use Option B. If they ask CI/CD, use Option C (Amplify + GitHub).
