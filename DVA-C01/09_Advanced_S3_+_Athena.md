# Advanced S3 + Atehna

---

## S3 MFA Delete

- MFA forces user o generate a conde on a device before doing important
    operations on S3.

- To use MFA-Delete, enable Versioning on S3 bucket.
- Need MFA to:
    - perma delete an object version.
    - suspend versioning on the bucket.
- Do not need MFA to:
    - enabling versioning.
    - listing deleted versions.

- Only the bucket owner (root account) can enable/disable MFA-Delete.
- MFA-Delete currently can only be enabled using the CLI.

    > aws s3api put-bucket-versioning --bucket <bucket-name> --versioning-configuration Status=enalbed,MFADelete=Enabled --mfa <arn-of-root-mfa-device> --profile <use-root-credentials>

---

## S3 Default Encrpytion vs Bucket Policies

- The old way to enable default encryption was to use a bucket policy to refuse
    any HTTP command wothout proper headers.

- The new way is to simply use the "default encryption" option in S3.
- Note that bucket policies are evalustaed before default encryption.

---


