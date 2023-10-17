+++
title = "static website"
date = "2023-10-12"
+++

# wtf is zola
[zola](https://www.getzola.org/) is a single-executable application wriiten in rust. it integrates a wide array of features inlcuding sass compilation and syntax highlighting, to streamline development. the content of the website is authored in [common mark](https://commonmark.org/), a universally compatible markdown specification. 

## install zola 
im currently running [pop_os](https://pop.system76.com/) 22.04 LTS and since i do not use flatpak or snap i have to unpack the debian package from their [gh page](https://github.com/barnumbirr/zola-debian). this is pretty easy just visit the page and install the release that matches your systems architecture. run `wget https://github.com/barnumbirr/zola-debian/releases/download/v0.17.2-1/zola_0.17.2-1_amd64_bullseye.deb` to install the debian bullseye v0.17.2-1 package and unpack it by running `sudo dpkg -i zola_0.17.2-1_amd64_bullseye.deb` and now i can run the `zola` cmd on my machine anywhere. `zola init` will populate the current directory if it is empty with the exclusion of hidden files or `zola init <your-dir>` will populate the specified directory only if the directory is empty. (the --force flag can be passed to both cmds to populate the directory despite the files). /// [install](https://www.getzola.org/documentation/getting-started/installation/) page for everyone else

# cloudflare setup
create a [cloudflare](https://www.cloudflare.com/) account and register a new domain. we are going to use this domain name for our s3 bucket names. 

# aws setup 
an s3 bucket can be made directly from the s3 console but the [aws cli](https://aws.amazon.com/cli/) can create and configure an s3 bucket for static website hosting. we will use the [s3 mb](https://awscli.amazonaws.com/v2/documentation/api/latest/reference/s3/mb.html) cmd to create our sub/apex domain (sub: `aws s3 mb s3://www.<bucket-name>` apex: `aws s3 mb s3://<bucket-name>`). in the aws console go to the newly created subdomain bucket and in the properties tab enable static website hosting. make sure to point to the index.html file. go to the permissions tab and disable 'block all public access'. in the 'bucket policy' tab add [bucket-policy](@/posts/s3_post.md#bucket-policy) which allows access to the buckets resources (or else your pages will return a 403) and denies direct access to the s3 endpoint and only allows access to cloudflare servers by configuring the cloudflare [ip addresses](https://www.cloudflare.com/ips/) in our iam policies. go to iam > policies and in json create a [new iam policy](@/posts/s3_post.md#iam-policy) with your bucket name in the resources. then go to iam > users and create a new user with the attached policy for its permissions. select 'create access key' and choose cli for its permissions. 

## iam-policy 
```
{
    "Version": "2012-10-17",
	"Statement": [{
		"Sid": "AccessToWebsiteBuckets",
		"Effect": "Allow",
		"Action": [
            "s3:PutBucketWebsite",
			"s3:PutObject",
			"s3:PutObjectAcl",
			"s3:GetObject",
			"s3:ListBucket",
			"s3:DeleteObject"
		],
		"Resource": [
			"arn:aws:s3:::subdomain-bucket-name",
			"arn:aws:s3:::subdomain-bucket-name/*"
		]    
    }]
}
```

## bucket-policy
```
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Sid": "AddPerm",
            "Effect": "Allow",
            "Principal": "*",
            "Action": "s3:GetObject",
            "Resource": "arn:aws:s3:::bucket-name/*",
            "Condition": {
                "IpAddress": {
                    "aws:SourceIp": [
                        "173.245.48.0/20",
                        "103.21.244.0/22",
                        "103.22.200.0/22",
                        "103.31.4.0/22",
                        "141.101.64.0/18",
                        "108.162.192.0/18",
                        "190.93.240.0/20",
                        "188.114.96.0/20",
                        "197.234.240.0/22",
                        "198.41.128.0/17",
                        "162.158.0.0/15",
                        "104.16.0.0/12",
                        "172.64.0.0/13",
                        "131.0.72.0/22",
                        "2400:cb00::/32",
                        "2606:4700::/32",
                        "2803:f800::/32",
                        "2405:b500::/32",
                        "2405:8100::/32",
                        "2a06:98c0::/29",
                        "2c0f:f248::/32"
                    ]
                }
            }
        }
    ]
}
```

# configure cloudflare
log into cloudflare and visit the dashboard to select the domain used to name the sub/apex domain buckets. go to dns > records and select the 'add a record' button in the 'dns management' box. set the type as `CNAME`, set name to `www`, and set the target as the s3 endpoint from the subdomain bucket with the exclusion of `https://` at the beginning then hit save. create a new record and set the type to cname, set the name value as your apex domain name e.g.: `fwvain.faith`, and finally set the value as the s3 endpoint from our apex domain bucket also with the exclusion of `https://` and hit save. since our s3 endpoint is not secured by tls we need to ensure our ssl/tls setting is set to `flexible` under ssl/tls > overview.  

# gh actions 
"i use gh actions so i can manic post quicker with automatic deployments on merges to main" - sum1. we need to set our repo secrets which is in the settings tab under the 'secrets and variables' dropdown. select 'actions' and 'new repository secret'. add `AWS_ACCESS_KEY_ID` and `AWS_SECRET_ACCESS_KEY` using the keys we obtained from the previously created iam user. add `S3_BUCKET` with your subdomain bucket name as the value and finally add `AWS_DEFAULT_REGION` with the region your bucket is in. we now need to setup a new github action [workflow](@/posts/s3_post.md#gh-workflow), this workflow [checks-out](https://github.com/actions/checkout) your repository, uses [wget](https://linux.die.net/man/1/wget) to fetch the linux release of zola version 0.17.2 and download it, unpackages it using [tar](https://linux.die.net/man/1/tar) we use the `-x` flag to extract the files // the `-z` flag to filter the archive using gzip since its in the .gz file format // the `-f` flag to specify the file we are unpacking, runs the newly created `./zola build` cmd to build the 'public/' directory, and finally uses the [sync](https://docs.aws.amazon.com/cli/latest/reference/s3/sync.html) cmd to sync our public/ directory with our subdomain bucket that utilizes the secrets we created earlier to deploy the public directory to our s3 bucket which will now be hosting our static website!  

## gh-workflow
```
name: Build and Publish to AWS
on:
  push:
    branches:
      - main
jobs:
  run:
    runs-on: ubuntu-latest
    timeout-minutes: 10
    steps:
      - name: checkout repo
        uses: actions/checkout@v4
        
      - name: install zola
        run: |
            wget https://github.com/getzola/zola/releases/download/v0.17.2/zola-v0.17.2-x86_64-unknown-linux-gnu.tar.gz
            tar -xzf zola-v0.17.2-x86_64-unknown-linux-gnu.tar.gz 

      - name: build 
        run: ./zola build
        
      - name: deploy to s3 bucket
        run: |
            aws s3 sync ./public s3://${{ secrets.S3_BUCKET }}
        env:
            AWS_ACCESS_KEY_ID: ${{ secrets.AWS_ACCESS_KEY_ID }}
            AWS_SECRET_ACCESS_KEY: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
            AWS_DEFAULT_REGION: ${{ secrets.AWS_DEFAULT_REGION }}
```
<!-- if you copy from zola site it is missing a comma and will flag a syntax error 
 - add new user specific for the iam role and gh-actions allows for website access of the bucket
 - setup actions secrets in github with new cli user 
 - add cloudflare instructions and change gh actions workflow file example 
 -->
# continue ?
ensure the `base_url` is set in the config toml to the cloudflare domain if it is not the inline linking will be wrong when zola builds the website. future plans: to add a cloudflare cdn for static content and ssl certificate creation.   
