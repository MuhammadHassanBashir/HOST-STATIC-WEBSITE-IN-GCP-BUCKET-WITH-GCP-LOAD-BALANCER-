# HOST-STATIC-WEBSITE-IN-GCP-BUCKET-WITH-GCP-LOAD-BALANCER-
HOST STATIC WEBSITE IN GCP BUCKET WITH GCP LOAD BALANCER AND ATTACH SSL CERTIFICATE


## WEBSITE LINK

    https://cloud.google.com/storage/docs/hosting-static-website#console

## Hosting a Static Website Using Google Cloud

    This guide provides a step-by-step tutorial on how to configure a Google Cloud Storage bucket to host a static website for a domain you own. Static websites use client-side technologies like HTML, CSS, and JavaScript but do not support server-side scripting.
    
    Since Cloud Storage alone does not provide HTTPS support for custom domains, we’ll use Cloud Storage with an external Application Load Balancer to enable HTTPS. You can still serve content over HTTP without a load balancer.

## Objectives:

- Upload and share your website files.
- Set up a load balancer and an SSL certificate.
- Connect your load balancer to your Cloud Storage bucket.
- Point your domain to the load balancer using an A record.
- Test the website.

## Costs Involved:

This tutorial uses the following Google Cloud services:

- Cloud Storage
- Cloud Load Balancing
- You can monitor potential charges by reviewing Google Cloud's billing tips.

## Prerequisites:

Before starting, ensure you have:

- A Google Cloud project with billing enabled.
- The Compute Engine API enabled.
- IAM roles for Storage Admin and Compute Network Admin.
- A domain you own (or manage).
- Cloud Storage bucket for your website files (if you don't have one, create it).
- You should also have some basic files for your website, including an index.html and a 404.html page.
- (Optional) If your Cloud Storage bucket shares the same name as your domain, verify domain ownership via Cloud Domains.

## Create Your HTML Files

First, create the index.html and 404.html files. These will be used for your static website.

## index.html:

    <!DOCTYPE html>
    <html lang="en">
    <head>
        <meta charset="UTF-8">
        <meta name="viewport" content="width=device-width, initial-scale=1.0">
        <title>Welcome to My Website</title>
        <style>
            body { font-family: Arial, sans-serif; text-align: center; padding: 50px; }
            h1 { color: #333; }
        </style>
    </head>
    <body>
        <h1>Welcome to My Static Website</h1>
        <p>This website is hosted on Google Cloud Storage.</p>
    </body>
    </html>

## 404.html:

    <!DOCTYPE html>
    <html lang="en">
    <head>
        <meta charset="UTF-8">
        <meta name="viewport" content="width=device-width, initial-scale=1.0">
        <title>404 - Page Not Found</title>
        <style>
            body { font-family: Arial, sans-serif; text-align: center; padding: 50px; }
            h1 { color: red; }
        </style>
    </head>
    <body>
        <h1>404 - Page Not Found</h1>
        <p>Sorry, the page you're looking for doesn't exist.</p>
    </body>
    </html>

## Step 1: Upload Your Website Files

You can upload files to Cloud Storage via:

Google Cloud Console:

- Go to the Cloud Storage Buckets page in the Google Cloud Console.
- Select your bucket.
- Click Upload files and select the files to upload.

Command Line:
        
    gcloud storage cp Desktop/index.html gs://my-static-assets

Terraform:

    resource "google_storage_bucket_object" "indexpage" {
      name         = "index.html"
      content      = "<html><body>Hello World!</body></html>"
      content_type = "text/html"
      bucket       = google_storage_bucket.static_website.id
    }

REST API (JSON):




