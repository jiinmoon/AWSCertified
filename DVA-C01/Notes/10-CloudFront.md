# CloudFront

- is a content delivery network.
- it is a localized cache at edge that improves read performance.
- currently there are 216 edge locations around globe.
- it provides DDoS protection, and integrates with WAF.
- can expose HTTPS and talks to internal HTTPS backends.

## CloudFront Origins

- The source of CloudFront can be either S3 or Custom HTTP.

- S3 Bucket:
    - used to distribute files and caching them at edge.
    - provide better security with CloudFront Origin Access Identity.
    - CloudFront can be used to upload files to S3.

- Custom Origin (HTTP)
    - Application Load Balancer
    - EC2 instance
    - S3 static website
    - etc.

## CloudFront Geo-Restriction

- Can restrict who can access your distribution with allow/deny lists of
  conturies.

- Useful to enforce copyright laws.

## Origin Access Identity

- OAI can be created and associated with an user trying to access distribution
  via CloudFront on S3.

- The associated CloudFront OAI has to be specified within the Bucket Policy.

## Cache

- We can cache based on headers, session cookies, or query string parameters.

- Cache lives at the CloudFront edge locations.
- We control TTL of the caches (0 to 1 year) by the origin using the
  Cache-Control header and Expires header.
- We can also invalidate the part of the cache using the `CreateInvalidation`.

- One strategy is to cache the static contents - it will not have headers nor
  session caching rules since the content is static and can be cached
  indefinitely.

- For dynamic content, we can fine control which objects to cache using the
  correcy headers and cookies. Or perhaps invalidate cache if newer object is
  updated by the origin.

## CloudFront Security

- You can restrict via Geo-location.
- We can enforce HTTPS:
    - Viewer protocol policy to reirect HTTP to HTTPS or HTTPS only.
    - Origin protocol policy to use HTTPS only or match viewer's protocol to
      backend.
- Note that S3 static website buckets do not support HTTPS.

## CloudFront Signed URL / Signed Cookies

- We can distribute contents using CloudFront using signed URL that will be
  valid for specified period (few mins to years).

- Signed URL is used for single files; Cookies to multiple files.

- Compared to S3 Pre-signed URLs:
    - CloudFront Signed URL allow access to a path regardless of the origin.
    - Account wide key-pair; only root can manage it.
    - Can filter by IP, path, date, expiration.
    - Can leverage on cache.

