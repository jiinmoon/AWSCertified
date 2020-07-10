AWS CloudFront
==============

- CloudFront is a Content Delivery Network (CDN).
- It is a cache location at the edge, which improves read performance.
    - When a file is requested, it is first served from the origin; but the
      edge location close to the user will cache the file. Thus, when
      subsequent requests are made by nearby users, it can read from the edge
      locations instead.
- Currently, 216 edge locations.
- It gives DDoS protection, integration with Shield, AWS Web Application
  Firewall.
- Can expose external HTTPS and can talk to internal HTTPS backends.

---

CloudFront -- Origins
---------------------

- What can CDN serve?

- **S3 bucket**

    - for distributing files and caching them at the edge.
    - enhanced securtiy with CloudFront **Origin Access Identity (OAI)**.
    - CloudFront can be used as an ingress to upload files to S3.

- **Custom Origin (HTTP)**
    
    - Apllication Load Balancer
    - EC2 instance
    - S3 Website (static S3 website enabled)
    - Any HTTP backend

ClundFront at a high level
--------------------------

- Edge Locations are spreaded across the globe.
- They are connected to the origin.
- Client sends an HTTP request to Edge Location.
- Edge location forwards the request to your origin which includes *query
  strings* and *request headers*.
- Edge location will cache the response back from the origin; so, on subsequent
  requests, it can serve the cached file.

CloudFront -- S3 as an Origin
-----------------------------

- Through public www, users will send their request to the Edge. Edge relays
  the request to the Origin (S3 bucket) through private AWS network.

- **For the edge location to access S3 bucket, it requies _Origin Access
  Identity_ (OAI) and _S3 bucket policy_**.

CloudFront -- ALB or EC2 as an Origin
-------------------------------------

- EC2 Instances must be available to public through security group.
- Edge has to go through the security group - thus, use allowlist to accept
  requests from public IPs of Edge locations.


- Same applies to the ALB - its security group rules must allow for public
  access.
- However, in the backend, EC2 can be a private - allowing only access from the
  ALB.

CloudFront Geo Restriction
--------------------------

- Restrict who can access your distribution:

    - Whitelist: allow users to access your content if they are in one of the
      approved countries.
    - Blacklist: prevent users from accessing if they are in one of the banned
      countries.

- *Country* is determined using a 3rd party geo-IP database.

- Use cases would be to enforce copy-right laws.

CloundFront vs S3 Cross Region Replication
------------------------------------------

- CloudFront:
    - Global Edge network.
    - Files are cached for a TTL (~ a day).
    - **Great for static content that much be available everywhere**.

- S3 Cross Region Replication:
    - Must be setip for each region you want replication to happen.
    - Files are updated in near real-time.
    - Read only.
    - **Great for dynamic content that needs to be available at low-latency in
      few regions**.


