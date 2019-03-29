# Building a Website for Production on the Cloud ☁️

![alt text](media/Workshop1/architecture_large.png "Architecture")

# Introduction

This workshop will guide you through a full production Cloud Deployment of your own personal blog including domain registration, configuration, deployment options, HTTPS, performance and standard development practices. The primary focus is not writing HTML, CSS or JavaScript but integrating multiple tools, pipelines and configurations to create a seamless experience. This is applicable not only to website design but many areas.

>There are only two hard things in Computer Science: cache invalidation and naming things.
>
>-- Phil Karlton

## What are we building?

![alt text](media/Workshop1/website.png "Screenshot of final website")

### Static website

You will be generating a static website with `hugo`, a tool written in Go. `hugo` sites are fully extensible, they can be swapped out entirely for another generator or just plain old HTML, CSS and JavaScript.

- Hugo gives us an easy way to write templates in `markdown`, style them, then generate HTML posts/pages from the markdown formatting. Markdown is a plain text authoring syntax to create documents like the current document you are viewing. This file is written in flavour of Markdown for github.com.
- `hugo` is part of the the JAM stack paradigm which gives us flexibility, open APIs and decoupling between content authoring, styling and deployment.

You will then use your domain name provider (namecheap.com) to point to a cached version of your website on the Cloudflare Content Delivery Network (CDN). This will result in global distribution of your website and improved load times. Your final website will be hosted on Google Cloud Platform, utilising Github with CircleCI for a Continuous Deployment/Integration process that pushes code through the Google Cloud SDK and uploads the resulting build files to Google Cloud Storage automatically.

# Prerequisites 
Complete this before the workshop if you can, it will save time! Some of these instructions may differ depending on operating system.

- Make sure you have a Credit/Debit Card you can use for Google Cloud sign up
- Registrations:
  - [NameCheap](https://namecheap.com) Domain provider or a free [.me](https://nc.me) domain!
  - [Cloudflare](https://cloudflare.com) CDN & Extra DNS provider
  - [Google Cloud Platform](https://console.cloud.google.com/freetrial)
    - Make sure you enable billing and add your card
  - [GitHub](https://github.com) (Hosting & building your website content)
  - [CircleCI](https://circleci.com) (Our CI/CD tool that integrates with github.com) make sure you link this to your github.com account
- Download and install:
  - [Hugo](https://gohugo.io/getting-started/installing) a static site generator with markdown templating 
  - [Google Cloud SDK](https://cloud.google.com/sdk/install) To access Google Storage and APIs
  - [Git](https://git-scm.com/downloads) (To manage github.com repositories and track code)

# Content
1. [Creating our Website Content](#💻-Creating-our-Website-Content)
2. [Google Cloud Platform - Cloud Hosting Provider](#☁️-Google-Cloud-Platform-Cloud-Hosting-Provider)
3. [Domain Setup and DNS config](#🌎-Domain-Setup-and-DNS-config)
4. [Email Setup](#✉️-Email-Setup)
5. [Cloudflare - Content Delivery Network (CDN)](#⚡-Cloudflare-Content-Delivery-Network-(CDN))
6. [Automatic Deployment Continued](#Automatic-Deployment-Continued)
7. [Extensions](#Extensions)

# 💻 Creating our Website Content

## Why do we want a static website?
- A static website is primarily HTML, CSS and JS and is usually fully interactive on the client side
- This may include routing, local state, animation and all manner of other cool things!
- However, we do not store state to databases, manage sessions or authorisations
- Sometimes the line is blurred between static and dynamic with new models of Serverless and other third party libraries

### To host and run a website we have a number of options:
- Run a server ourselves (A bit vintage but fun!)
- Upload your website content to another server managed elsewhere
- Use a fully managed solution such as Wordpress/Wix
- Use a static hosting solution (No server side scripting or code!)

## Getting started

1. Login to github.com and `fork` the workshop repository using the button on the right, this will copy the repository into your own github.com account

![alt text](media/Workshop1/fork_repo.png "Forking")

2. Now run a `git clone <url>` using the repository you forked in your own github. This will download the repository to the directory you are in.

![alt text](media/Workshop1/clone_with_https.png "Clone")

1. Once cloned `cd` into the repository and run `hugo new site bens-blog` this will create a new directory for your hugo project called `bens-blog`
2. Find a theme you like! Currently we are using [hugo-theme-even](https://github.com/olOwOlo/hugo-theme-even) if you don't like this you can find another but it may require a different setup!
3. Open your blog directory eg `bens-blog` and open the `themes` dir
4. To add a theme we will be using: `git clone https://github.com/olOwOlo/hugo-theme-even /even`
5. Once cloned, copy `even/exampleSite/config.toml` (our site configuration) to the `bens-blog` dir and replace the default `config.toml`
6. Go back to `bens-blog` and create a new blog post page by running `hugo new post/i_like_static_sites.md`
7. Edit your post at: `content/post/i_like_static_sites.md` If you don't know markdown look here: https://www.markdownguide.org/basic-syntax/

Add something like the following:
```
  1 ---
  2 title: "I_like"
  3 date: 2019-03-29T21:50:00+11:00
  4 draft: true
  5 ---
  6
  7 # Hello welcome to my blog
  8
  9 Thanks for joining
```

1.  Now try and run your web server `hugo server -D` Your server will now be running at: http://localhost:1313
2.  Don't stop the server and make some changes to your post. You should see it hot reload.
3.  Create more blog posts and pages relevant to your wishes or skip if you wish to get into the nitty gritty deployment
4.  Run `hugo` to build all your assets including files marked as drafts
5.  Check the `bens-blog/public` directory to see you all the asset files/folder created from our previous command
6.  Files in `public` are now the self contained build and will be served from Google Cloud Storage in the next steps.
7.  You should also be able to open `bens-blog/public/index.html` and see your website without running a server

**Be careful you may need configure the base url correctly in the `config.toml` file**
- If you are serving from example.com you will need to add this into the `config.toml` file under the `baseUrl` setting

## 📚 Resources
The Jamstack paradigm - https://jamstack.wtf  
A list of awesome static site generators - https://www.staticgen.com  
A list of awesome headless content management systems for the jamstack - https://headlesscms.org

# ☁️ Google Cloud Platform - Cloud Hosting Provider

![alt text](media/Workshop1/bucket_creation.png "Screenshot of final design")

## Basics

There are a number of ways to host our website. For this workshop we will be using Google Cloud Platform due to the ease of use and free credit/trial period provided. `Google Cloud Storage` is Google's offering for "bucket" storage (many Cloud offerings have a similar service) this is blob file storage with a selection of other features including metadata, permissions and redundancy. You can use it to store whatever you want.
  
To upload to GCS you can use the `gcloud` tool or the web interface. `gcloud` is a command line interface allowing us submit a number of powerful commands to the GCP APIs and manage infrastructure.

## It's GCP time

1. Login to the [Google Cloud Console](https://console.cloud.google.com/) with your Google account
2. Make sure you have enabled billing and your credit/debit card is added otherwise you can't create a project or use the service
3. **Optional:** Set billing alerts for something like $10 so you don't waste the credit.
4. Create a new project for example: `bens-blog`
5. Create a bucket under storage with a name like: `bens-blog`
6. Select `permissions` and update permissions allowing `allUsers` access. This will allow anyone access to this bucket and give the role `Storage Object Viewer` this means anyone can access the content of the bucket. Be careful what you upload from now on.

![alt text](media/Workshop1/add_Users.png "GCP permissions")

7. Make sure you have generated your website files using `hugo` 
8. Then upload all files in the `bens-blog/public` directory to this bucket
9. Go back to bucket browser and find the public link to your file. It will be similar to: https://storage.googleapis.com/bens-blog/index.html where `bens-blog` is your bucket name chosen before.

![alt text](media/Workshop1/public_access.png "Forking")

10. Make sure everything is displayed correctly. You may need to change the config.toml file `baseURL` or set your permissions correctly on the bucket.

```
baseUrl = "storage.googleapis.com/bens-blog/"  
title = "Choose an awesome title"
```

## Starting to get Automated

To start automation we should understand how to automatically build and upload our website content.

1. Make sure you have the `GCloud SDK` downloaded and test using `gcloud help`
2. Before running any `gcloud` tools you will need to login to the same Google account that you signed up to GCP
3.  Run: `gcloud auth login` this will open a browser window to login to Google.com
4. Once logged in make some changes to your basic website or add another blog post like before
5. Run `hugo` again to build everything into the output directory `public`
6. Now you can use `gcloud` to upload to our bucket without using the gui. Make sure you `cd` into the `bens-blog` dir and then run the command below with your bucket name you created before.
 ```
 gsutil rsync -R public gs://bucket-name
 ```
`gsutil` is the Google Storage Utility and allows you to manage GCS this command will do a recursive sync of the `public` dir to the GCS bucket and upload all files.

Now navigate to your bucket again, have a look and see if your files are there, you can access your site. It should be available at: storage.googleapis.com/bucket-name/index.html

Well done, onto our domain setup now!

## 🔥 Extra:
- Keep exploring the GCP console
- Look at alternative static site generators
- Keep digging into `hugo`

# 🌎 Domain Setup and DNS config

Time to make use of that `.me` domain or another domain on namecheap.com. We need to use the Domain Name System (DNS) to point our domain to the endpoint in Google Cloud Platform.

1. Login to [Google Search Console](https://search.google.com/search-console/welcome) with the **same** account you used to Signup for Google Cloud Platform. Here you will need to add your new fancy domain to the console so we can verify our ownership of the domain for GCP.

![alt text](media/Workshop1/domain_verify.png "Domain Verify")

1. Enter your domain name you have registered as a property
2. You will be presented with an option `Verify ownership via DNS record` copy that record so you can add it to namecheap.com DNS settings.
3. Log into namecheap.com go to `Advanced DNS` set up a new DNS record with the valid `TXT` record with `TTL = 1min` to confirm your ownership of the domain.

```
NAME        TYPE     DATA
@           TXT      google-site-verification=12356abcdef....
```

You should also add a `CNAME` record which points your domain to Google. If you are wondering what the `@` is it means the root domain will point to Google Storage. If you put `test` it would end up being a subdomain like test.yourdomain.com

```
NAME        TYPE     DATA
@           CNAME    c.storage.googleapis.com
```

4. Now go back to the search console and try to re-verify your ownership, it may take a little while but should verify! This means you can now link your domain with the GCS storage bucket
5. Log back into Google Cloud Platform and try create a new bucket with the name of your domain eg. `yourdomain.com` if verification was successful you should be able to do this.
6.  Try it out, you may need to wait a little bit for the DNS settings to propagate again
7. If you get any errors like below you may have some config issues check your permissions, bucket name etc.
8. If all is well your website should now be resolved through yourdomain.com

```
<Error>
<Code> </Code>
<Message> </Message>
</Error>
```

## 🔥 Extra:
- Explore some of the other DNS types, try creating a subdomain eg. blog.yourdomain.com
- Multiple buckets can be created for different subdomains

# ⚡ Cloudflare - Content Delivery Network (CDN)

You have finished creating your website, it's up and running on your new domain but you want speed and reliability

You did a test on [cdnplanet](https://www.cdnplanet.com/tools/cdnperfcheck) and your friends in the Europe and Philippines are getting slow response times 😢
To speed up content delivery you will need to cache files closer to your end users. The server might be located in Australia but you can cache globally over a large network. A CDN will intelligently route traffic to those edge locations so users can reduce latency thus speeding up load times.

Make sure you modify the records in namecheap.com to match the required DNS settings by cloudflare.com

1. Login in to your cloudflare.com account and obtain the new nameservers
2. You will be prompted to enter in your domain you own
3. Once entered you will get a similar info page as below

![alt text](media/Workshop1/cloudflare_setup.png "Screenshot of final design")

4. Log back into namecheap.com and update your nameservers 
5. Under the `Domain` settings in the dashboard for your domain change the `Nameservers` setting to the above `Custom` nameservers for cloudflare
6. This may take quite a while to complete so you may only have you Cloudflare setup after you have left this workshop
7. Verify the settings are correct and confirm
8. `Important` move onto the next section then come back come back here
9. Go back to cloudflare.com and check the verification
10. If you are verified retry your domain using [cdnplanet](https://www.cdnplanet.com/tools/cdnperfcheck)
11. You should see improved results!

## 🔥 Extra:
- Cloudflare has a huge amount of options both free and paid, explore all the tabs!

# 🏗️ Automatic Deployment Continued

We can make use of Github and TravisCI/CircleCI for our deployment toolkit. We will link the CI system with our repository on GitHub.

## Using CircleCI for auto deployment

1. Create a `.circleci` folder in the root directory of the repository
2. Inside this directory create the following content in a `config.yml` file where `blog-name` is the directory in your repo and `domain.com` is your chosen domain, or the standard bucket if you don't have the domain

```
version: 2
jobs:
  build:
    docker:
      - image: google/cloud-sdk

    steps:
      - checkout

      - run: |
          echo $GCLOUD_SERVICE_KEY | gcloud auth activate-service-account --key-file=-
          gcloud --quiet config set project ${GOOGLE_PROJECT_ID}
          gsutil rsync -R blog-name/public gs://domain.com
```

1. Login to circleci.com using github and `Follow` the repository to give CircleCI access to it.
2. Make sure you are inside the repository 
3. Run `git status` to see status of files
4. Run `git add` to add all files that you need
5. Make sure you commit your changes `git commit -m "Finished dev"`
6. Push your changes using `git push`
7. This will push all changes you have commited to github.com
8. As long as the `.circleci/config.yml` is present then cirlceci.com will pick up the repository and start trying to run the above
9. Wait for a push, you will see a red failure this means circleci.com is not authorised to access your GCS bucket and upload content


![alt text](media/Workshop1/ci_build_example.png "Build Status")

### For further info:  
We will be authorizing CircleCI access to push code changes to our Google Cloud Platform bucket.
https://circleci.com/docs/2.0/google-auth/


1. Make sure you follow this: https://cloud.google.com/sdk/docs/authorizing#authorizing_with_a_service_account
2. Log back into GCP
3. Go to `Service Accounts` Tab https://console.cloud.google.com/iam-admin/serviceaccounts
4. Create a service account

![alt text](media/Workshop1/service_account.png "Create a service account")

5. Choose a name and description like `circleci` and `allows access to my-bucket`
6. Select `permissions` as `Storage Object Admin`
7. Skip user access
8. `Create key` then download the `json` file

The content will be similar to below:

```json
{
  "type": "service_account",
  "project_id": "bens-site-fb",
  "private_key_id": "12345",
  "private_key": "-----BEGIN PRIVATE KEY-----",
  "client_email": "circleci@....com",
  "client_id": "1234",
  "auth_uri": "https://accounts.google.com/o/oauth2/auth",
  "token_uri": "https://oauth2.googleapis.com/token",
  "auth_provider_x509_cert_url": "https://www.googleapis.com/oauth2/v1/certs",
  "client_x509_cert_url": "https://www.googleapis.com/robot/v1/metadata/x509/circleci%40bens-site-fb.iam.gserviceaccount.com"
}
```

13. Run `gcloud auth activate-service-account --key-file=file.json` where `file.json` is the file you downloaded
14. You should get a successful result `Activated service account credentials for...`
15. Once you have a service account add these environment variables to your build environment in CircleCI. where `GCLOUD_SERVICE_KEY` is the content of the json.
   
```
GCLOUD_SERVICE_KEY
GOOGLE_PROJECT_ID
```

![alt text](media/Workshop1/env_setup.png "Forking")

1. Once everything has been setup make some changes to your website code
```
hugo
git commit -m "amazing build"
git push
```

1. Now check CircleCI, if everything is setup correctly you should see a nice green tick and success!
2. If not you can just re-run the `build` in the CircleCI website and check error messages to make sure everything is setup correctly!
3.  Login to GCP and check the content of your bucket has it changed?
4.  Now we need to login to cloudflare.com to carry out cache invalidation and remove our old content from cache
5.  Find the cache invalidation section in cloudflare.com and try it, check your website has updated

## 🔥 Extra:
- Look into open source CI/CD platforms eg. Travis CI
- gitlab.com provides some great features including it's own CI/CD built in
- Setup your own more complicated CI/CD pipeline in GCP and make use of that credit

## 📚 Resources
Docker - docker.com  
CircleCI vs Travis vs Jenkins  
Atlassian [CI/CD principles](https://www.atlassian.com/continuous-delivery/principles/continuous-integration-vs-delivery-vs-deployment)


# Extensions

## ✉️ Email Setup

> To enable email forwarding in namecheap.com we will need to change the nameservers back to namecheap.com nameservers thus losing our ability to use the cloudflare.com CDN.

We now have our domain pointing to our website but what about emails. Don't you want to receive emails @ yourdomain.com?

We can go back to namecheap.com and enable `email forwarding` which will forward emails to our account used to signup to namecheap. We can set a `catch-all` or just have certain emails forward. Emails being sent won't be sent from this address but if you want to set that up as well check the extras section below.

1. Go to domain list
2. Manage
3. Email Forwarding

Set something up like the following:

![alt text](media/Workshop1/email_forwarding.png "Architecture")

Depending on whether you set a catch-all or not you should be able to send an email to the relevant address: eg. `<anything>@yourdomain.com`

## 🔥 Extensions:
- Read into DMARC email validation with DNS
- Setup your own email forwarding/filtering on AWS using SES
- Setup free sender emailing service so you can send emails from your domain eg. Received from me@awesomedomain.com rather than forwarding to your gmail.com email.
- Dynamic content
  - 3rd party (form submissions, comment sections)
  - Serverless functions for dynamically creating pages or sending an email
- Extra subdomains and features eg. help.yourdomain.com
- Switching the static site hosting to Amazon Web Services
- SEO
  - business.google.com
  - search.console.google.com
  
# Acknowledgements
https://github.com/olOwOlo/hugo-theme-even
