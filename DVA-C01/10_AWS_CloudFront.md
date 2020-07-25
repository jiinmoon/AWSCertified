AWS CloudFront
==============

CloudFront is a **Content Delivery Network (CDN)** which is a cache located at
the _edges_ to improve the read performance for users. For example, a request
for a file in S3 is made. On first run, it will be served directly from the
origin S3, but any subsequent requests for the same file near that edge
location can instead use the cache to retrieve the file.

It provides DDoS protection, integration with Shield and AWS Web Application
Firewall.

Can expose external HTTPS and talk to internal HTTPS backends.

CloudFront -- Origins
---------------------

Origins are what CloudFront serves.

**S3 Bucket**

- used to distribute files and caching them at the edge.
- enhanced security with CloudFront **Origin Access Identity (OAI)**.
- CloudFront can also be used to upload files to S3 bucket.

**Custom Origin (HTTP)**

- Application Load Balancer.
- EC2 Instances.
- S3 static website.
- Any HTTP backend.

CloudFront -- at high level
---------------------------

- Edge locations are spread across the globe and connected to the origin.
- Client sends an HTTP request to Edge Location.
- Edge location forwards the request to your origin which includes query srings
  and request headers.
- Edge location will cache the response from the origin.

CloudFront -- S3 as an Origin
-----------------------------

Through www, Client will snd their request to the Edge. Edge relays the request
to the S3 bucket through private AWS network.

**For edge locations to access S3 bucket, it reuqires _Origin Access Identity
(OAI) and _S3 Bucket Policy_**.

CloudFront -- ALB or EC2 as an Origin
-------------------------------------

EC2 instances must be available to public _through security group_. Edge has to
go through the secrutiy group to reach the instances; so, use allowlist to
accept requests from public IPs of Edge locations.

Same rule applies for ALB; but if the EC2 instance is the target group of the
ALB, the instance can be private - accepting only the traffics from the ALB.

CloudFront -- Geo Restriction
-----------------------------

Restrict who can access your distribution:

- Whitelist allows users to access if they are in approvd countries.
- Blacklist does oppositie.

Here, "country" is determined with a 3rd party geo-IP database. This is useful
for enforcing copyright laws.

CloudFront vs S3 Cross-Region Replication
-----------------------------------------

CloudFront

- Global Edge network.
- Files are caches for a TTL (usually <= a day).
- **Great for static content that needs to be available everywhere**.

S3 Cross-Region Replication

- Must setup in each region where the replication needs to take place.
- Files are updated in near real-time.
- **Read Only**.
- **Great for dynamic content that needs to be available at low-latency in
  certain regions**.

CloudFront -- Caching
---------------------

Caching is done based on following:

- Headers
- Session Cookies
- Query String Parameters

Cache resides at each CloudFront _Edge Locations_.

The goal is to maximize the number of _cache-hits_; to achieve this, we control
the TTL (0 - 1 year) which can be set by the origin using `Cache-Control` or
`Expires` header. You may invalidate part of the cache with
`CreateInvalidation` API.

Maximizing Cache-hits via Separating Static and Dynamic Distributions
---------------------------------------------------------------------

Static objects request require no headers nor session saching rules since we
can simply store them without having to worry about impacting the cache hit
rate.

Dynamic requests require more engineering on how to cache based on correct
headers and cokies.

_Note that longer TTL means increase in chance that users will be receiving
stale cached objects while the origin is updated until TTL expires_. To force
cache eviction, there will be a fee charged.

CloudFront -- Security
----------------------

**CloudFront Geo-Restriction**

- Restrict based on white/blacklist of countries.
- Used mostly to enforce the copyright laws.

**HTTPS**

- Viewer Protocol Policy:
    - Client to Edge Location.
    - Edge Location can redirect HTTP to HTTPS or accept HTTPS only.
    - This will enforce encryption on traffics to Edges.

- Origin Portocol Policy (to HTTP or S3):
    - HTTPS traffics only fro mEdge Location to Origin.
    - Or, match the viewer's protocol used.

CloudFront -- Signed URL and Signd Cookies
------------------------------------------

Suppose you wish to distribute paid shared content to premium users over the
world.

- With CloudFront Signed URL and Cookies, we can attach policy to enforce:
    - URL expiration date.
    - IP ranges (sources) that can access the data from.
    - Trusted signers - which AWS accounts can crate the signed urls?

- **Signed URL gives access to individual files**.
- **Signed Cookies give access to multiple files**.

CloudFront -- Signed URL vs S3 Pre-signed URL
---------------------------------------------

**CloudFront Signed URL**

- Allow access to a path regardless of the origin.
- Account wide key-pair and only root can manage it.
- Can filter based on IP, path, date, and expiration.
- Can leaverge caching features.

**S3 Pre-signed URL**

- Issue a request as a person who pre-signed the URL.
- Uses IAM key of the signing IAM principal.
- Limited lifetime.
