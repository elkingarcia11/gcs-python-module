# Google Cloud Storage Python Module

A comprehensive Python module for managing Google Cloud Storage operations using service account authentication.

## Features

- ✅ Create and delete buckets
- ✅ List all buckets in a project
- ✅ Upload files to buckets (with advanced options)
- ✅ Download files from buckets (with advanced options)
- ✅ List files in buckets
- ✅ Delete files from buckets
- ✅ Check bucket existence
- ✅ Get file metadata
- ✅ Concurrent upload and download (transfer manager)
- ✅ Service account authentication only
- ✅ Comprehensive error handling and logging
- ✅ Environment variable support
- ✅ Type hints for better development experience

## Installation

1. Clone this repository:

```bash
git clone <repository-url>
cd gcs-python-module
```

2. Install dependencies:

```bash
pip install -r requirements.txt
```

3. Set up Google Cloud service account:

   - Go to Google Cloud Console > IAM & Admin > Service Accounts
   - Create a new service account or select an existing one
   - Create a new key (JSON format)
   - Download the JSON key file to a secure location

4. Configure environment variables:

```bash
cp env.example .env
# Edit .env with your actual values
```

5. Set the service account credentials:

```bash
export GOOGLE_APPLICATION_CREDENTIALS="/path/to/your/service-account-key.json"
```

6. Add "/path/to/your/service-account-key.json" to your .gitignore file

## Quick Start

```python
from gcs_client import GCSClient

gcs = GCSClient(service_account_path="/path/to/service-account-key.json")

# List all buckets
buckets = gcs.list_buckets()
print(f"Available buckets: {buckets}")

# Upload a file
success = gcs.upload_file("my-bucket", "local-file.txt")
print(f"Upload: {'Success' if success else 'Failed'}")

# List files in bucket
files = gcs.list_files("my-bucket")
print(f"Files in bucket: {files}")
```

## Usage Examples

### Using the GCSClient Class

```python
from gcs_client import GCSClient

gcs = GCSClient(service_account_path="/path/to/service-account-key.json")

# Create a bucket
# (Requires Storage Admin permissions)
gcs.create_bucket("my-bucket", location="US")

# Upload a file with custom name and advanced options
gcs.upload_file(
    "my-bucket",
    "local-file.txt",
    destination_blob_name="remote-file.txt",
    content_type="text/plain",
    predefined_acl="public-read",
    timeout=120,
    checksum='md5'
)

# Download a file with advanced options
gcs.download_file(
    "my-bucket",
    "remote-file.txt",
    "downloaded-file.txt",
    timeout=120,
    checksum='crc32c'
)

# List files with prefix
files = gcs.list_files("my-bucket", prefix="data/")

# Get file metadata
metadata = gcs.get_file_metadata("my-bucket", "remote-file.txt")
if metadata:
    print(f"File size: {metadata['size']} bytes")

# Delete a file
gcs.delete_file("my-bucket", "remote-file.txt")

# Delete bucket (with force to remove all files)
gcs.delete_bucket("my-bucket", force=True)
```

### Concurrent Upload and Download (Transfer Manager)

```python
# Upload many files concurrently
filenames = ["file1.txt", "file2.txt", "file3.txt"]
gcs.upload_many_from_filenames(
    bucket_name="my-bucket",
    filenames=filenames,
    source_directory="/local/dir",
    blob_name_prefix="uploads/",
    max_workers=4
)

# Download many files concurrently
blob_names = ["uploads/file1.txt", "uploads/file2.txt"]
gcs.download_many_to_path(
    bucket_name="my-bucket",
    blob_names=blob_names,
    destination_directory="downloads/",
    max_workers=4
)
```

## API Reference

### GCSClient Class

#### Constructor

```python
GCSClient(service_account_path: Optional[str] = None, log_level: int = logging.INFO)
```

#### Methods

- `create_bucket(bucket_name: str, location: str = "US") -> bool`
- `delete_bucket(bucket_name: str, force: bool = False) -> bool`
- `list_buckets() -> List[str]`
- `upload_file(bucket_name: str, source_file_path: str, destination_blob_name: Optional[str] = None, content_type: Optional[str] = None, predefined_acl: Optional[str] = None, if_generation_match: Optional[int] = None, if_generation_not_match: Optional[int] = None, if_metageneration_match: Optional[int] = None, if_metageneration_not_match: Optional[int] = None, timeout: int = 60, checksum: str = 'auto', retry: Optional[object] = None) -> bool`
- `download_file(bucket_name: str, source_blob_name: str, destination_file_path: str, start: Optional[int] = None, end: Optional[int] = None, raw_download: bool = False, if_etag_match: Optional[str] = None, if_etag_not_match: Optional[str] = None, if_generation_match: Optional[int] = None, if_generation_not_match: Optional[int] = None, if_metageneration_match: Optional[int] = None, if_metageneration_not_match: Optional[int] = None, timeout: int = 60, checksum: str = 'auto', retry: Optional[object] = None) -> bool`
- `list_files(bucket_name: str, prefix: str = "") -> List[str]`
- `delete_file(bucket_name: str, blob_name: str) -> bool`
- `bucket_exists(bucket_name: str) -> bool`
- `get_file_metadata(bucket_name: str, blob_name: str) -> Optional[dict]`
- `upload_many_from_filenames(bucket_name: str, filenames: List[str], source_directory: str = "", blob_name_prefix: str = "", skip_if_exists: bool = False, blob_constructor_kwargs: Optional[dict] = None, upload_kwargs: Optional[dict] = None, threads: Optional[int] = None, deadline: Optional[float] = None, raise_exception: bool = False, worker_type: str = "process", max_workers: int = 8, additional_blob_attributes: Optional[dict] = None) -> List`
- `download_many_to_path(bucket_name: str, blob_names: List[str], destination_directory: str = "", blob_name_prefix: str = "", download_kwargs: Optional[dict] = None, threads: Optional[int] = None, deadline: Optional[float] = None, create_directories: bool = True, raise_exception: bool = False, worker_type: str = "process", max_workers: int = 8, skip_if_exists: bool = False) -> List`

## Environment Variables

Create a `.env` file with the following variables:

```env
# Google Cloud Project ID
GOOGLE_CLOUD_PROJECT_ID=your-project-id-here

# Path to your Google Cloud service account key file (REQUIRED)
# Download this from Google Cloud Console > IAM & Admin > Service Accounts
GOOGLE_APPLICATION_CREDENTIALS=path/to/your/service-account-key.json
```

## Service Account Setup

1. **Create a Service Account:**

   - Go to Google Cloud Console
   - Navigate to IAM & Admin > Service Accounts
   - Click "Create Service Account"
   - Give it a name and description

2. **Assign Permissions:**

   - Add the following roles:
     - Storage Admin (for full access)
     - Storage Object Admin (for object operations only)
     - Storage Object Viewer (for read-only access)

3. **Create and Download Key:**

   - Click on the service account
   - Go to Keys tab
   - Click "Add Key" > "Create new key"
   - Choose JSON format
   - Download the key file

4. **Set Environment Variable:**
   ```bash
   export GOOGLE_APPLICATION_CREDENTIALS="/path/to/your/service-account-key.json"
   ```

## Advanced Features

### Class-Level Logging

Each GCSClient instance has its own logger with configurable log levels:

```python
# Create client with debug logging
gcs_debug = GCSClient(log_level=logging.DEBUG)

# Create client with only warnings and errors
gcs_quiet = GCSClient(log_level=logging.WARNING)
```

### Permissions Note

Some operations (like creating or deleting buckets, uploading files) require specific permissions for your service account. If your service account lacks these permissions, the module will log a warning or error and continue gracefully where possible.

### Error Handling

All methods return `False` or `None` on failure and log the error. Check logs for details if an operation fails.

---

For more details, see the code and docstrings in `gcs_client.py`.
