+++
title = "s3 static website with zola"
date = "2022-01-01"
+++

## wtf is zola

[zola](https://www.getzola.org/) is a single-executable application wriiten in rust. it integrates a wide array of features inlcuding sass compilation and syntax highlighting, to streamline development. the content of the website is authored in [common mark](https://commonmark.org/), a universally compatible markdown specification. 

## install zola 
im currently running [pop_os](https://pop.system76.com/) 22.04 LTS and since i do not use flatpak or snap i have to unpack the debian package from their [gh page](https://github.com/barnumbirr/zola-debian). this is pretty easy just visit the page and install the release that matches your systems architecture. i installed the bullseye v0.17.2-1 package and unpacked it by running `sudo dpkg -i zola_0.17.2-1_amd64_bullseye.deb` and now i can run the `zola` cmd on my machine anywhere. `zola init` will populate the current directory if it is empty with the exclusion of hidden files or `zola init <your-dir>` will populate the specified directory only if the directory is empty. (the --force flag can be passed to both cmds to populate the directory despite the files)

## create s3 bucket from aws cli 
test


<!-- if you copy from zola site it is missing a comma and will flag a syntax error 
 - add s3 mb cmds also include website endpoint 
 - add custom iam role permissions in code block
 - duckdns docker compose  
 - add new user specific for the iam role and gh-actions allows for website access of the bucket
 - setup actions secrets in github with new cli user -->







