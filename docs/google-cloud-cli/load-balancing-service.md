# Google Cloud CLI - Managing Load Balancing Service

When you configure the load balancing service, your virtual machine instances receives packets that are destined for the static external IP address you configure. Instances made with a Compute Engine image are automatically configured to handle this IP address.

!!! Note

    **Note:** Learn more about how to set up Network Load Balancing from the [Backend service-based external passthrough Network Load Balancer overview][gcp-backend-service-based-external-passthrough-network-load-balancer-overview]{:target="_blank"} guide.

# Create a static external IP address for your load balancer

```shell
gcloud compute addresses create network-lb-ip-1 \
--region Region
```

**Output**:

```shell
Created [https://www.googleapis.com/compute/v1/projects/qwiklabs-gcp-03-xxxxxxxxxxx/regions//addresses/network-lb-ip-1].
```

# Add a legacy HTTP health check resource

```shell
gcloud compute http-health-checks create basic-check
```

# Create the target pool and forwarding rule

A target pool is a group of backend instances that receive incoming traffic from external passthrough NLBs. All backend instances of a target pool must reside in the same Google Cloud region.

Run the following to create the target pool and use the health check, which is required for the service to function:

```shell
gcloud compute target-pools create www-pool \
--region Region --http-health-check basic-check
```

# Add instances created earlier to the pool

```shell
gcloud compute target-pools add-instances www-pool \
--instances www1,www2,www3
```

Next you'll make the [forwarding rule][gcp-forwarding-rule]{:target="_blank"}. A forwarding rule specifies how to route network traffic to the backend services of a load balancer.

# Add a forwarding rule to the pool:

```shell
gcloud compute forwarding-rules create www-rule \
--region  Region \
--ports 80 \
--address network-lb-ip-1 \
--target-pool www-pool
```

# Send traffic to your instances

Now that the load balancing service is configured, you can start sending traffic to the forwarding rule and watch the traffic be dispersed to different instances.

1. Enter the following command to view the external IP address of the www-rule forwarding rule used by the load balancer:

```shell
gcloud compute forwarding-rules describe www-rule --region Region
```

2. Access the external IP address:

```shell
IPADDRESS=$(gcloud compute forwarding-rules describe www-rule --region Region --format="json" | jq -r .IPAddress)
```

3. Show the external IP address:

```shell
echo $IPADDRESS
```

4. Use the `curl` command to access the external IP address, replacing `IP_ADDRESS` with an external IP address from the previous command:

```shell
while true; do curl -m1 $IPADDRESS; done
```

The response from the `curl` command alternates randomly among the three instances. If your response is initially unsuccessful, wait approximately 30 seconds for the configuration to be fully loaded and for your instances to be marked healthy before trying again.

[gcp-backend-service-based-external-passthrough-network-load-balancer-overview]: https://cloud.google.com/compute/docs/load-balancing/network/