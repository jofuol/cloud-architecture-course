# **Google Cloud CLI - Managing region and zone**

!!! Information

    Learn more about choosing zones and regions in Compute Engine's [Regions and zones documentation][gcp-regions-zones-doc]{:target="_blank"}.

# Set the default region for all resources

```shell
gcloud config set compute/region Region
```

# Set the default zone for all resources

```shell
gcloud config set compute/zone Zone
```

[gcp-regions-zones-doc]: https://cloud.google.com/compute/docs/zones