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

    curl -X POST --data-binary @index.html \
      -H "Content-Type: text/html" \
      -H "Authorization: Bearer $(gcloud auth print-access-token)" \
      "https://storage.googleapis.com/upload/storage/v1/b/my-static-assets/o?uploadType=media&name=index.html"

## Step:2 Share your files

To make all objects in your bucket readable to anyone on the public internet:

In the Google Cloud console, go to the Cloud Storage Buckets page.

- Go to Buckets
- In the list of buckets, click the name of the bucket that you want to make public.
- Select the Permissions tab near the top of the page.
- If the Public access pane reads Not public, click the button labeled Remove public access prevention and click Confirm in the dialog that appears.
- Click the add_box Grant access button.
- The Add principals dialog box appears.
- In the New principals field, enter allUsers.
- In the Select a role drop down, select the Cloud Storage sub-menu, and click the Storage Object Viewer option.
- Click Save.
- Click Allow public access.

Once shared publicly, a link icon appears for each object in the public access column. You can click this icon to get the URL for the object.

To learn how to get detailed error information about failed Cloud Storage operations in the Google Cloud console, see Troubleshooting.

If wanted, you can alternatively make portions of your bucket publicly accessible.

Visitors receive a http 403 response code when requesting the URL for a non-public or non-existent file. See the next section for information on how to add an error page that uses a http 404 response code.

## Step 3: Recommended: assign specialty pages

You can assign an index page suffix and a custom error page, which are known as specialty pages. Assigning either is optional, but if you don't assign an index page suffix and upload the corresponding index page, users who access your top-level site are served an XML document tree containing a list of the public objects in your bucket.

For more information on the behavior of specialty pages, see Specialty pages.

Google Cloud Console:

In the Google Cloud console, go to the Cloud Storage Buckets page.

- Go to Buckets
- In the list of buckets, find the bucket you created.
- Click the Bucket overflow menu (more_vert) associated with the bucket and select Edit website configuration.
- In the website configuration dialog, specify the main page and error page.
- Click Save.

Command Line:

Use the buckets update command with the --web-main-page-suffix and --web-error-page flags.

In the following sample, the MainPageSuffix is set to index.html and NotFoundPage is set to 404.html:

    gcloud storage buckets update gs://my-static-assets --web-main-page-suffix=index.html --web-error-page=404.html

If successful, the command returns:

    Updating gs://www.example.com/...
      Completed 1

REST API JSON:

Have gcloud CLI installed and initialized, which lets you generate an access token for the Authorization header.

Create a JSON file that sets the mainPageSuffix and notFoundPage properties in a website object to the desired pages.

    In the following sample, the mainPageSuffix is set to index.html and notFoundPage is set to 404.html:
    
    
    
    {
      "website":{
        "mainPageSuffix": "index.html",
        "notFoundPage": "404.html"
      }
    }

Use cURL to call the JSON API with a PATCH Bucket request. For the bucket my-static-assets:

    curl -X PATCH --data-binary @web-config.json \
      -H "Authorization: Bearer $(gcloud auth print-access-token)" \
      -H "Content-Type: application/json" \
      "https://storage.googleapis.com/storage/v1/b/my-static-assets"

## STEP 4: Set up your load balancer and SSL certificate

Cloud Storage doesn't support custom domains with HTTPS on its own, so you also need to set up an SSL certificate attached to an HTTPS load balancer to serve your website through HTTPS. This section shows you how to add your bucket to a load balancer's backend and how to add a new Google-managed SSL certificate to the load balancer's frontend.

Start your configuration

In the Google Cloud console, go to the Load balancing page.

- Go to Load balancing
- Click Create load balancer.
- For Type of load balancer, select Application Load Balancer (HTTP/HTTPS) and click Next.
- For Public facing or internal, select Public facing (external) and click Next.
- For Global or single region deployment, select Best for global workloads and click Next.
- For Load balancer generation, select Classic Application Load Balancer and click Next.
- Click Configure.

The configuration window for your load balancer appears.

- Basic configuration
- Before continuing with the configuration, enter a Load balancer name, such as staticwebsite-lb.

Configure the frontend

- This section shows you how to configure the HTTPS protocol and create an SSL certificate. You can also select an existing certificate or upload a self-managed SSL certificate.

Click Frontend configuration.

- (Optional) Give your frontend configuration a Name.
- For Protocol, select HTTPS (includes HTTP/2).
- For IP version, select IPv4. If you prefer IPv6, see IPv6 termination for additional information.
- For the IP address field:
- In the dropdown, click Create IP address.
- In the Reserve a new static IP address pop-up, enter a name, such as example-ip for the Name of the IP address.
- Click Reserve.

Note: Reserving a new static IP address incurs additional costs if the IP address is not attached to a forwarding rule. To avoid such costs, delete the static IP address when you delete the associated load balancer.

- For Port, select 443.
- In the Certificate field dropdown, select Create a new certificate. The certificate creation form appears in a panel. Configure the following:
- Give your certificate a Name, such as example-ssl.
- For Create mode, select Create Google-managed certificate.
- For Domains, enter your website name, such as www.example.com. If you want to serve your content through additional domains such as the root domain example.com, press Enter to add them on additional lines. Each - - certificate has a limit of 100 domains.
- Click Create.

(Optional) If you want Google Cloud to automatically set up a partial HTTP load balancer for redirecting HTTP traffic, select the checkbox next to Enable HTTP to HTTPS redirect.

- Click Done.

Configure the backend

- Click Backend configuration.
- In the Backend services & backend buckets dropdown, click Create a backend bucket.
- Choose a Backend bucket name, such as example-bucket. The name you choose can be different from the name of the bucket you created earlier.
- Click Browse, found in the Cloud Storage bucket field.
- Select the my-static-assets bucket you created earlier, and click Select.
- (Optional) If you want to use Cloud CDN, select the checkbox for Enable Cloud CDN and configure Cloud CDN as desired. Note that Cloud CDN may incur additional costs.
- Click Create.

Configure routing rules

- Routing rules are the components of an external Application Load Balancer's URL map. For this tutorial, you should skip this portion of the load balancer configuration, because it is automatically set to use the backend you just configured.

 Review the configuration

- Click Review and finalize.
- Review the Frontend, Routing rules, and Backend.
- Click Create.

You may need to wait a few minutes for the load balancer to be created.

Connect your domain to your load balancer

- After the load balancer is created, click the name of your load balancer: example-lb. Note the IP address associated with the load balancer: for example, 30.90.80.100. To point your domain to your load balancer, create an A record using your domain registration service. If you added multiple domains to your SSL certificate, you must add an A record for each one, all pointing to the load balancer's IP address. For example, to create A records for www.example.com and example.com:


    NAME                  TYPE     DATA
    www                   A        30.90.80.100
    @                     A        30.90.80.100
    See Troubleshooting domain status for more information about connecting your domain to your load balancer.

Recommended: Monitor the SSL certificate status

- It might take up to 60-90 minutes for Google Cloud to provision the certificate and make the site available through the load balancer. To monitor the status of your certificate:

With Console:

- Go to the Load balancing page in the Google Cloud console.
- Go to Load balancing
- Click the name of your load balancer: example-lb.
- Click the name of the SSL certificate associated with the load balancer: example-ssl.
- The Status and Domain status rows show the certificate status. Both must be active in order for the certificate to be valid for your website.

With Command:

To check the certificate status, run the following command:

    gcloud compute ssl-certificates describe CERTIFICATE_NAME \
      --global \
      --format="get(name,managed.status)"

To check the domain status, run the following command:

    gcloud compute ssl-certificates describe CERTIFICATE_NAME \
      --global \
      --format="get(managed.domainStatus)"

See Troubleshooting SSL certificates for more information about certificate status.

Test the website

- Once the SSL certificate is active, verify that content is served from the bucket by going to https://www.example.com/test.html, where test.html is an object stored in the bucket that you're using as the backend. If you set the MainPageSuffix property, https://www.example.com goes to index.html.

Clean up

- After you finish the tutorial, you can clean up the resources that you created so that they stop using quota and incurring charges. The following sections describe how to delete or turn off these resources.

Delete the project

- The easiest way to eliminate billing is to delete the project that you created for the tutorial.

To delete the project:

Caution: Deleting a project has the following effects:
- Everything in the project is deleted. If you used an existing project for the tasks in this document, when you delete it, you also delete any other work you've done in the project.
Custom project IDs are lost. When you created this project, you might have created a custom project ID that you want to use in the future. To preserve the URLs that use the project ID, such as an appspot.com URL, delete selected resources inside the project instead of deleting the whole project.

If you plan to explore multiple architectures, tutorials, or quickstarts, reusing projects can help you avoid exceeding project quota limits.

- In the Google Cloud console, go to the Manage resources page.
- Go to Manage resources
- In the project list, select the project that you want to delete, and then click Delete.
- In the dialog, type the project ID, and then click Shut down to delete the project.
- Delete the load balancer and bucket

If you don't want to delete the entire project, delete the load balancer and bucket that you created for the tutorial:

- Go to the Load balancing page in the Google Cloud console.
- Go to Load balancing
- Select the checkbox next to example-lb.
- Click Delete.
- (Optional) Select the checkbox next to the resources you want to delete along with the load balancer, such as the my-static-assets bucket or the example-ssl SSL certificate.
- Click Delete load balancer or Delete load balancer and the selected resources.
- Note: If you only want to delete the bucket you created, follow the instructions at Deleting buckets.
- Release a reserved IP address
- To delete the reserved IP address you used for the tutorial:

- In the Google Cloud console, go to the External IP addresses page.

Go to External IP addresses

- Select the checkboxes next to example-ip.
- Click Release static address.
- In the confirmation window, click Delete.


## DEPLOYMENT AND LEARNING STEP

Deployment of Static Website on Google Cloud with Application Load Balancer and SSL

I was tasked with deploying a static website for the development team, who had created a GCP bucket named ask-angie-frontend containing a build folder with static content. My objective was to host this content on Google Cloud with proper HTTPS routing and an SSL certificate using an Application Load Balancer.

**Initial Steps: Making the Bucket Public**

To begin, I needed to ensure that the bucket was publicly accessible. Unfortunately, the Google Cloud Console wasn't providing clear information on the bucket's public status. To verify this, I used the following command:

    gsutil iam get gs://ask-angie-frontend

The output confirmed that the bucket wasn't public. To resolve this, I made the bucket publicly accessible using the following command (this can also be done via the Google Cloud Console UI):

    gcloud storage buckets add-iam-policy-binding gs://ask-angie-frontend --member=allUsers --role=roles/storage.objectViewer

After re-running **gsutil iam get gs://ask-angie-frontend**, I confirmed that the bucket was now public, as the allUsers member was granted the **roles/storage.objectViewer** role. I also verified this through the Cloud Console.

**Configuring the Application Load Balancer**

With the bucket publicly accessible, the next step was to configure the Application Load Balancer (ALB) to serve the content. I handled the following aspects of the load balancer setup:

1. Frontend Configuration (SSL & IP Reservation)

    First, I reserved a static IP for the load balancer.
    I created a Google-managed SSL certificate and configured HTTP to HTTPS redirection, ensuring secure traffic distribution.

2. Backend Configuration
    
    I created a backend bucket in the load balancer configuration, linking it to the ask-angie-frontend bucket, which contained the static content.
    I selected the build folder as the location of the content, setting it as the source for the load balancer’s backend.

3. Host and Path Rules

    Since the static content was located in the build folder (and not the root), I needed to configure host and path rules. This was crucial because the static content, including the index.html file, wasn't directly accessible from the root path.
    
    I set up the following rule:
    Host: askangie.disearch.ai (the domain through which traffic would be routed)
    Path: /build/* (to point traffic to the build folder)
    Backend: The backend bucket I had configured earlier.
    Alternatively, to route all traffic to the bucket, I set the path rule as /*.

**DNS Configuration and Testing**
    
    Once the load balancer was configured, I obtained the load balancer's IP address. I updated the domain's DNS settings in Google Cloud DNS, pointing askangie.disearch.ai to the load balancer IP.

    After waiting for SSL certificate provisioning, I tested the domain in the browser. The content was served, but I noticed that the index.html page wasn't accessible directly and required the full path (https://askangie.disearch.ai/build/index.html).

**Resolving Folder Structure Issue**

    A colleague suggested moving the contents of the build folder to the root of the bucket for better accessibility. I followed these steps to achieve this:

1- Backup the build Folder Locally: I took a backup of the build folder before making any changes, just to ensure no data was lost:

    gsutil -m cp -r gs://ask-angie-frontend/build /home/hassan/Project/hosting-static-website/build

2- Move Content to Bucket Root: I moved the contents of the build folder to the root of the bucket using this command:

    gsutil -m mv gs://ask-angie-frontend/build/* gs://ask-angie-frontend/

I verified the move by listing the contents of the bucket with gsutil ls and also checked via the Google Cloud Console.

3- Set index.html as Default: To ensure that visiting the domain would automatically load the index.html page, I configured the bucket's website settings:

    In the Google Cloud Console, I navigated to the Buckets section.
    I selected the ask-angie-frontend bucket and clicked on Edit website configuration.
    I specified index.html as the MainPageSuffix and saved the settings.

**Final Testing**

    After all the configurations, I re-tested the domain, and it successfully routed traffic to the index.html page without requiring the full path. The static website was now fully functional with HTTPS, and the load balancer was efficiently managing secure traffic distribution.


