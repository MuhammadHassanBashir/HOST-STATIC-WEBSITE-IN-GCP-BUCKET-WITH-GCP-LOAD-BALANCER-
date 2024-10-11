# HOST-STATIC-WEBSITE-IN-GCP-BUCKET-WITH-GCP-LOAD-BALANCER-
HOST STATIC WEBSITE IN GCP BUCKET WITH GCP LOAD BALANCER AND ATTACH SSL CERTIFICATE


## WEBSITE LINK

    https://cloud.google.com/storage/docs/hosting-static-website#console

## Hosting a Static Website Using Google Cloud

This guide provides a step-by-step tutorial on how to configure a Google Cloud Storage bucket to host a static website for a domain you own. Static websites use client-side technologies like HTML, CSS, and JavaScript but do not support server-side scripting.

Since Cloud Storage alone does not provide HTTPS support for custom domains, we’ll use Cloud Storage with an external Application Load Balancer to enable HTTPS. You can still serve content over HTTP without a load balancer.

Objectives:

Upload and share your website files.
Set up a load balancer and an SSL certificate.
Connect your load balancer to your Cloud Storage bucket.
Point your domain to the load balancer using an A record.
Test the website.

Costs Involved:

This tutorial uses the following Google Cloud services:

    Cloud Storage
    Cloud Load Balancing
    You can monitor potential charges by reviewing Google Cloud's billing tips.

Prerequisites:

Before starting, ensure you have:

    A Google Cloud project with billing enabled.
    The Compute Engine API enabled.
    IAM roles for Storage Admin and Compute Network Admin.
    A domain you own (or manage).
    Cloud Storage bucket for your website files (if you don't have one, create it).
    You should also have some basic files for your website, including an index.html and a 404.html page.

    (Optional) If your Cloud Storage bucket shares the same name as your domain, verify domain ownership via Cloud Domains.

