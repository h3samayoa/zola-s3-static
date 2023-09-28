+++
title = "s3 static website with zola"
date = "2023-09-28"
+++

## wtf is zola

[zola](https://www.getzola.org/) is a single-executable application wriiten in rust. it integrates a wide array of features inlcuding sass compilation and syntax highlighting, to streamline development. the content of the website is authored in [common mark](https://commonmark.org/), a universally compatible markdown specification. 

## install zola 
im currently running [pop_os](https://pop.system76.com/) 22.04 LTS and since i do not use flatpak or snap i have to unpack the debian package from their [gh page](https://github.com/barnumbirr/zola-debian). this is pretty easy just visit the page and install the release that matches your systems architecture. i installed the bullseye v0.17.2-1 package and unpacked it by running `sudo dpkg -i zola_0.17.2-1_amd64_bullseye.deb` and now i can run the `zola` cmd on my machine anywhere. `zola init` will populate the current directory if it is empty with the exclusion of hidden files or `zola init <your-dir>` will populate the specified directory only if the directory is empty. (the --force flag can be passed to both cmds to populate the directory despite the files)

## aws stuff
an s3 bucket can be made directly from the console but the [aws cli](https://aws.amazon.com/cli/) can create and configure an s3 bucket for static website hosting. the [s3 mb](https://awscli.amazonaws.com/v2/documentation/api/latest/reference/s3/mb.html) cmd will create a new bucket `aws s3 mb s3://<bucket-name>`. in the aws console go to the newly created s3 bucket and in the properties tab enable static website hosting. make sure to point to the index.html file. go to the permissions tab and disable 'block all public access'. in the 'bucket policy' tab add [bucket-policy](@/posts/s3_post.md#bucket-policy) which allows access to the buckets resources or else your pages will return a 403. go to iam > policies and in JSON create a [new policy](@/posts/s3_post.md#iam-policy) with your bucket name. then go to iam > users and create a new user with the attached policy for its permissions. select 'create access key' and choose cli for its permissions. 

### iam-policy 

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
			"arn:aws:s3:::Bucket-Name",
			"arn:aws:s3:::Bucket-Name/*"
		]    
    }]
}
```

### bucket-policy
```
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Sid": "AddPerm",
            "Effect": "Allow",
            "Principal": "*",
            "Action": "s3:GetObject",
            "Resource": "arn:aws:s3:::Bucket-Name/*"
        }
    ]
}
```

<!-- if you copy from zola site it is missing a comma and will flag a syntax error 
 - add s3 mb cmds also include website endpoint 
 - add custom iam role permissions in code block
 - duckdns docker compose  
 - add new user specific for the iam role and gh-actions allows for website access of the bucket
 - setup actions secrets in github with new cli user -->







