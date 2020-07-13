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

CloundFront Caching
-------------------

- Cache based on following:
    - Headers
    - Session Cookies
    - Query String Parameters

- The cache is located at each CloudFront **Edge Location**.

- Client makes a request to Edge Location; it checks for whether requested
  object is in cache, update appropriately based on Headers and Cookies.
    - Cache miss? request is made forward to Origin.

- _Goal is to maximize the cache hit rate_.
- To do this, we control the TTL (0 sec to 1 yr), which can be set by the
  origin using `Cache-Control` or `Expires` header.

- Invalidate part of the cache using `CreateInvalidation` API.

Maximize Cache-hits via Separating Static v Dynamic Distributions
-----------------------------------------------------------------

- Static object requests require no headers/session caching rules since we can
  simply cache whatever we grab, and this will maximize the cache hit.

- Dynamic requests need more thought on how to cache based on correct headers
  and cookies.

- Note that longer TTL increases the possibility that the users will be seeing
  the older versions of the cached objects even when origin is updated, until
  the TTL expires.

CloudFront Security
-------------------

**CloudFront Geo Restriction**
- A way to restrict who can access your distribution.
    - Whitelist/Blacklist the countries.
    - _country_ is determined by 3rd party Geo-IP database.
    - useful for Copyright laws.

**HTTPS**
- Viewer Protocol Policy:
    - Client to Edge Location.
    - redirect HTTP to HTTPS or use HTTPS only.
    - enforces encryption for traffics.
- Origin Protocol Polcy (HTTP or S3):
    - HTTPS only from Edge Location to Origin.
    - Match viewer (use which ever protocol used).

- Note that S3 bucket static websites do not support HTTPS.

CloundFront Signed URL / Signed Cookies
---------------------------------------

- i.e. you want to distribute paid shared content to premium users over the
  world.

- CloudFront Signed URL / Cookie; we can attach a policy with:
    - URL expiration
    - IP ranges to access the data from
    - trusted signers (which AWS accounts can create signed URLs)

- How long should the URL be valid for?
    - Shared content (movie, music) - make it short ~ mins.
    - Private content (private to user) - make it last for years.

- _Signed URL gives access to individual files - one signed URL per file._
- _Signed Cookeis give access to multiple files - one signed Cookie for many
  files_. 

- i.e. suppose we have CloudFront set up with multiple Edge locations connect
  to our S3 bucket origin via OAI - meaning objects can only be accessed via
  CloudFront. Client wishes to access the object, thus will send the request to
  an authentication/authorization application. The app will use AWS SDK to
  generate the signed URL. Client then uses signed URL to request the object
  from CloudFront.

CloudFront Signed URL vs S3 Pre-Signed URL
------------------------------------------

- CloudFront Signed URL:
    - Allow access to a path, no matter the origin.
    - Account wide key-pair; only root can manage it.
    - Can filter by IP, path, date, expiration.
    - Can leverage caching features.

- S3 Pre-Signed URL:
    - Issue a request as the person who pre-signed the URL.
    - Uses the IAM key of the signing IAM principal.
    - Limited lifetime.


