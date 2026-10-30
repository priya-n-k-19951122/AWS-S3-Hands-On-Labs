# AWS S3 Hands-On Labs

This repository contains documentation and reference notes for practical labs conducted on Amazon Simple Storage Service (S3). The labs cover core S3 features including access control, versioning behaviors, static website hosting, and Cross-Region Replication (CRR).

---

## Lab 1: Basic Bucket Operations & Public Access

### Prerequisites & Setup
1. **Create Source Bucket:** Created a general-purpose S3 bucket (e.g., `demopnkbkt` in the `ap-south-1` Mumbai region).
2. **Object Ownership:** Enabled **ACLs (Access Control Lists)** during or after bucket creation.
3. **Block Public Access (BPA):** Unchecked **Block *all* public access** to allow public hosting capabilities.

### Uploading & Testing Objects
1. Uploaded sample assets (`devopsimage.png` and `middleware_admin.html`) to the bucket root.
2. **Granting Access:** Selected the uploaded object, went to **Actions** -> **Edit ACL**, and explicitly granted **Read** permissions to the public (Everyone).
3. **Verification:** Copied the **Object URL** from the properties page and pasted it into a browser to confirm the asset loads successfully without an `Access Denied` error.

---

## Lab 2: S3 Bucket Versioning & Lifecycle States

Bucket versioning functions like a recycling bin, protecting against accidental deletions or overwrites by preserving history.

### Workflow Verification
1. **Enable Versioning:** Navigated to the bucket **Properties** tab and toggled **Bucket Versioning** to **Enabled**.
2. **Testing File Overwrites:** Uploaded a file, made modifications locally, and uploaded the exact same file again. 
3. **Toggling View:** Enabled the **Show versions** switch in the AWS Console to inspect the unique Version IDs assigned to each iteration.

### Understanding Delete Markers
* **Soft Deletion:** When versioning is active and an object is deleted normally, S3 places a **Delete Marker** on top of the object stack. The object seems gone from the standard view, but it is not permanently erased.
* **Restoring Objects:** To retrieve the original file, toggle **Show versions**, select the row containing the **Delete marker**, click **Delete**, and choose **Delete permanently**. The previous active version automatically becomes visible again.

---

## Lab 3: Static Website Hosting & Redirection

Configured an S3 bucket to act as an automated HTTP landing page or web forwarder.

### Scenario A: Hosting a Static Site
1. Navigated to **Properties** -> **Static website hosting** -> **Edit**.
2. Selected **Enable** and chose **Host a static website**.
3. Set the **Index document** to `index.html` and saved changes.
4. Used the generated **Bucket website endpoint** URL to view the webpage over HTTP.

### Scenario B: Redirecting Requests to a Third-Party Domain
1. Inside the **Static website hosting** configuration, changed the hosting type to **Redirect requests for an object**.
2. **Host Name Configuration:** Set the target hostname strictly to the domain (e.g., `://swiggy.com`). 
   * *Note: Do not prepend the domain name with protocol identifiers like `http://` or `https://` in the hostname field.*
3. **Protocol Selection:** Explicitly selected **https** under the standalone protocol dropdown menu.
4. **Testing:** Visited the raw `http://...s3-website...` endpoint string in an incognito window to verify instant routing redirection.

---

## Lab 4: Cross-Region Replication (CRR)

Implemented an automated, asynchronous backup mechanism between two distinct geographic cloud locations.

### Architecture Map
```text
[ Source Bucket ]                    [ Destination Bucket ]
Region: ap-south-1 (Mumbai)   --->   Region: ca-central-1 (Canada)
Versioning: Enabled                  Versioning: Enabled
```

### Configuration Implementation Steps
1. **Target Setup:** Created a completely empty target destination bucket in a separate AWS region (e.g., `ca-central-1` Central Canada).
2. **Alignment Rule:** Explicitly enabled **Bucket Versioning** on the new destination bucket to match the source criteria.
3. **Replication Rule Blueprint:**
   * Returned to the source bucket (`demopnkbkt`) and opened the **Management** tab.
   * Under **Replication rules**, clicked **Create replication rule**.
   * Defined a rule name and set the scope to **Apply to all objects in the bucket**.
   * Set the **Destination** by browsing S3 and selecting the target Canada bucket.
   * Under **IAM Role**, chose **Create new role** to authorize S3 to carry out secure cross-region write operations automatically.
4. **Synchronizing Existing Data:** Upon saving the rule, opted **Yes, replicate existing objects** to launch an S3 Batch Operations job, executing an immediate catch-up sync for items present prior to rule activation.
