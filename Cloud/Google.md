
# Google Cloud Storage Links

• **Question**: Where can we get the google cloud link?  
   **Answer**: We might find cloud links in the source code or javascript files, or if you open an image uploaded to the cloud, the cloud link might appear for you.

• The Google Cloud Storage link usually looks like this: `https://storage.googleapis.com/<Bucket_name>/<File_name>`
   - Bucket link (no specific file): `https://storage.googleapis.com/my_bucket/`
   - Direct file link: `https://storage.googleapis.com/my_bucket/images/logo.png`

• It could also look like this: `https://storage.googleapis.com/username-dev/index.html` or `dev-username`. Here, `username` is the bucket name and is unique for each user, while `dev` represents the environment variable (modifiable), whereas `index.html` is the active file/key being served. The file `index.html` is called the key in this context. You can FUZZ it and discover more things.

## Case (1): Testing the bucket for public access and exploiting it if accessible:

- Rather than accessing `https://storage.googleapis.com/username-dev/index.html`, we'll remove the file and access `https://storage.googleapis.com/username-dev` directly to observe the response.
- In case we get an **AccessDenied (Error)** response, it indicates that access to the bucket or file is restricted. Therefore, we can verify permissions using the `testPermissions` API endpoint:

  ```bash
  https://www.googleapis.com/storage/v1/b/BUCKET_NAME/iam/testPermissions?permissions=storage.buckets.delete&permissions=storage.buckets.get&permissions=storage.buckets.getIamPolicy&permissions=storage.buckets.setIamPolicy&permissions=storage.buckets.update&permissions=storage.objects.create&permissions=storage.objects.delete&permissions=storage.objects.get&permissions=storage.objects.list&permissions=storage.objects.update
  ```

- This link is an API endpoint for testing permissions on a Google Cloud Storage bucket, checking multiple permissions including:
    - Bucket operations (delete/get/update)
    - IAM policy operations
    - Object operations (create/delete/get/list/update)
    - The `BUCKET_NAME` would be replaced with an actual bucket name when used.
  
- Once you access this link, it will display the permissions you have. The permissions you might find include:
  - `storage.buckets.get` | `storage.buckets.list` | `storage.buckets.update` | `storage.buckets.delete` | `storage.buckets.create` | `storage.buckets.getIamPolicy` | `storage.buckets.setIamPolicy`
    - `get`: Allows you to read the content of files in the bucket.
      - `gsutil cp gs://bucketname-dev/file.txt .`     # Download a specific file from the bucket to your machine.
      - `gsutil rsync gs://bucketname-dev/ .`         # Download all files from the bucket to your machine.
    - `list`: Allows you to view the list of files in the bucket (but not read their content).
      - `gsutil ls gs://bucketname-dev/`              # List all the files in the bucket.
    - `update`: Allows you to modify files or bucket settings.
      - `gsutil cp localfile.txt gs://bucketname-dev/existingfile.txt`
    - `delete`: Allows you to delete files from the bucket.
      - `gsutil rm gs://bucketname-dev/filetodelete.txt`
    - `create`: Allows you to create or upload new files to the bucket.
      - `gsutil cp localfile.txt gs://bucketname-dev/`
    - `getIamPolicy`: Allows you to view the IAM policy of the bucket (who has what permissions).
      - `gsutil iam get gs://bucketname-dev`
    - `setIamPolicy`: Allows you to modify the IAM policy, meaning you can change access permissions for the bucket.
      - `gsutil iam ch allUsers:admin gs://bucketname-dev`     # All users will have the same permissions as the admin.
      - `gsutil iam set policy.json gs://bucketname-dev`       # See the following line to understand why we use a JSON file.

- **gsutil iam set**: If you want to apply a more specific policy, such as granting one user reader permissions and another writer permissions, you would have a `policy.json` file like:

  ```json
  {
    "bindings": [
      {
        "role": "roles/storage.objectViewer",
        "members": ["user:example@example.com"]
      },
      {
        "role": "roles/storage.objectAdmin",
        "members": ["user:admin@example.com", "user:your-email@example.com"]
      }
    ]
  }
  ```

  And you would run:
  ```bash
  gsutil iam set policy.json gs://bucketname-dev
  ```
  This would apply the policy defined in `policy.json`.

## Case (2): Fuzzing:

- In case the `bucketname-dev` has restrictions, we will perform FUZZ on `dev`, and we might swap the words as follows: `dev-bucketname`, and also perform FUZZ on `dev` to try to access the files.

## How can you tell if this is a vulnerability?

→ If you can list files, for example, or copy files from the server to your machine, upload files from your machine to the server, or modify user permissions, all of these are considered vulnerabilities. Their severity varies depending on the impact of each.
