# Google Cloud CLI - Managing Virtual Machines

# Create a Virtual Machine Instance

```shell
gcloud compute instances create www1 \
--zone=Zone \
--tags=network-lb-tag \
--machine-type=e2-small \
--image-family=debian-11 \
--image-project=debian-cloud \
--metadata=startup-script='#!/bin/bash
apt-get update
apt-get install apache2 -y
service apache2 restart
echo "
<h3>Web Server: www1</h3>" | tee /var/www/html/index.html'
```

This command create a virtual machine:

- With name `www1`.
- In your default zone.
- With the `network-lb-tag` network tag.
- Using the `e2-small` machine type.
- Using the `debian-11` image from the `debian-cloud` project.
- And configure the instance to run a startup script that installs and starts the Apache web server and sets a custom HTML message on the default web page.

## List your instances

```shell
gcloud compute instances list
```