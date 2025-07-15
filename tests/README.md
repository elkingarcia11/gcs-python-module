# Integration Tests for Google Cloud Storage Python Module

This directory contains integration tests for the `GCSClient` class that interact with actual Google Cloud Storage buckets and files.

## Prerequisites

Before running these tests, ensure you have:

1. **Google Cloud Storage credentials**: A service account JSON file with appropriate permissions
2. **Required packages**: Install dependencies from `requirements.txt`
3. **Test bucket access**: The service account should have read/write access to create and manage test buckets

## Setup

### 1. Create and Activate Virtual Environment

```bash
# Create virtual environment
python -m venv venv

# Activate virtual environment
# On macOS/Linux:
source venv/bin/activate

# On Windows:
# venv\Scripts\activate
```

### 2. Install Dependencies

```bash
# Ensure pip is up to date
pip install --upgrade pip

# Install dependencies
pip install -r requirements.txt
```

### 3. Configure Credentials

Set up your Google Cloud credentials using one of these methods:

#### Option A: Service Account (Recommended for CI/CD)

1. Place your service account credentials file in the project root as `service_account_credentials.json`
2. Set the environment variable:
   ```bash
   export GOOGLE_APPLICATION_CREDENTIALS="service_account_credentials.json"
   ```

#### Option B: Application Default Credentials (ADC)

1. Set up ADC as described in [Google Cloud documentation](https://cloud.google.com/docs/authentication/external/set-up-adc)
2. Set your project ID:
   ```bash
   export GOOGLE_CLOUD_PROJECT_ID="your-google-cloud-project-id"
   ```

#### Option C: Environment Variables

Set both credentials and project ID:

```bash
export GOOGLE_APPLICATION_CREDENTIALS="path/to/your/service-account.json"
export GOOGLE_CLOUD_PROJECT_ID="your-google-cloud-project-id"
```

### 4. Verify Installation

```bash
# Check installed packages
pip list

# Verify GCS client can be imported
python -c "from gcs_client import GCSClient; print('GCSClient imported successfully')"
```

## Running Tests

### Run All Integration Tests

**Important**: Make sure your virtual environment is activated before running tests.

```bash
# Verify venv is active (you should see (venv) in your prompt)
# If not active, run: source venv/bin/activate

python -m pytest tests/integration_test.py -v
```

### Run Individual Tests

You can run each test function individually using pytest with the `-k` flag or by specifying the exact test method:

#### Method 1: Using pytest -k flag

```bash
# Test bucket creation only
python -m pytest tests/integration_test.py -k "test_create_bucket" -v

# Test file upload only
python -m pytest tests/integration_test.py -k "test_upload_file" -v

# Test multiple specific functions
python -m pytest tests/integration_test.py -k "test_create_bucket or test_list_buckets" -v
```

#### Method 2: Using exact test method path

```bash
# Test bucket creation
python -m pytest tests/integration_test.py::TestIntegrations::test_create_bucket -v

# Test listing buckets
python -m pytest tests/integration_test.py::TestIntegrations::test_list_buckets -v

# Test bucket existence check
python -m pytest tests/integration_test.py::TestIntegrations::test_bucket_exists -v

# Test file upload
python -m pytest tests/integration_test.py::TestIntegrations::test_upload_file -v

# Test listing files
python -m pytest tests/integration_test.py::TestIntegrations::test_list_files -v

# Test file download
python -m pytest tests/integration_test.py::TestIntegrations::test_download_file -v

# Test getting file metadata
python -m pytest tests/integration_test.py::TestIntegrations::test_get_file_metadata -v

# Test file deletion
python -m pytest tests/integration_test.py::TestIntegrations::test_delete_file -v

# Test bucket deletion
python -m pytest tests/integration_test.py::TestIntegrations::test_delete_bucket -v
```

#### Method 3: Using unittest directly

```bash
# Run a specific test method
python -c "
import unittest
from tests.integration_test import TestIntegrations

# Create test suite with only the test you want
suite = unittest.TestSuite()
suite.addTest(TestIntegrations('test_create_bucket'))
runner = unittest.TextTestRunner(verbosity=2)
runner.run(suite)
"
```

## Test Functions Explained

### 1. `test_create_bucket`

**What it tests**: Creates a new bucket in Google Cloud Storage

- **Function tested**: `GCSClient.create_bucket()`
- **Expected behavior**: Returns `True` if bucket creation succeeds, `False` otherwise
- **Test bucket name**: `test-bucket-integration`
- **Dependencies**: Requires GCS write permissions

### 2. `test_list_buckets`

**What it tests**: Lists all buckets in the GCS project

- **Function tested**: `GCSClient.list_buckets()`
- **Expected behavior**: Returns a list of bucket names (non-empty)
- **Dependencies**: Requires GCS read permissions

### 3. `test_bucket_exists`

**What it tests**: Checks if a specific bucket exists

- **Function tested**: `GCSClient.bucket_exists()`
- **Expected behavior**: Returns `True` if the test bucket exists
- **Test bucket name**: `test-bucket-integration`
- **Dependencies**: Requires the test bucket to exist (run `test_create_bucket` first)

### 4. `test_list_files`

**What it tests**: Lists all files in a specific bucket

- **Function tested**: `GCSClient.list_files()`
- **Expected behavior**: Returns a list of file names in the bucket
- **Test bucket name**: `test-bucket-integration`
- **Dependencies**: Requires the test bucket to exist and contain files

### 5. `test_upload_file`

**What it tests**: Uploads a local file to GCS bucket

- **Function tested**: `GCSClient.upload_file()`
- **Expected behavior**: Returns `True` if upload succeeds
- **Test file**: `env.example` (local file)
- **Dependencies**: Requires the test bucket to exist and the local file to be present

### 6. `test_download_file`

**What it tests**: Downloads a file from GCS bucket to local storage

- **Function tested**: `GCSClient.download_file()`
- **Expected behavior**: Returns `True` if download succeeds
- **Test file**: `env.example` (file in bucket)
- **Dependencies**: Requires the file to exist in the bucket (run `test_upload_file` first)

### 7. `test_get_file_metadata`

**What it tests**: Retrieves metadata for a file in GCS

- **Function tested**: `GCSClient.get_file_metadata()`
- **Expected behavior**: Returns a dictionary with file metadata (name, size, content_type, etc.)
- **Test file**: `env.example`
- **Dependencies**: Requires the file to exist in the bucket

### 8. `test_delete_file`

**What it tests**: Deletes a file from GCS bucket

- **Function tested**: `GCSClient.delete_file()`
- **Expected behavior**: Returns `True` if deletion succeeds
- **Test file**: `env.example`
- **Dependencies**: Requires the file to exist in the bucket

### 9. `test_delete_bucket`

**What it tests**: Deletes a bucket from GCS

- **Function tested**: `GCSClient.delete_bucket()`
- **Expected behavior**: Returns `True` if deletion succeeds
- **Test bucket name**: `test-bucket-integration`
- **Dependencies**: Requires the bucket to be empty (run `test_delete_file` first)

## Test Execution Order

For a complete test run, the tests should be executed in this order due to dependencies:

1. `test_create_bucket` - Creates the test bucket
2. `test_bucket_exists` - Verifies bucket creation
3. `test_upload_file` - Uploads test file
4. `test_list_files` - Lists files in bucket
5. `test_get_file_metadata` - Gets file metadata
6. `test_download_file` - Downloads the file
7. `test_delete_file` - Deletes the file
8. `test_delete_bucket` - Deletes the bucket

## Troubleshooting

### Common Issues

1. **Virtual environment not activated**: Ensure you see `(venv)` in your terminal prompt
2. **Authentication errors**: Ensure your service account has the necessary permissions
3. **Credentials not found**: Check that `GOOGLE_APPLICATION_CREDENTIALS` environment variable is set correctly
4. **Project ID not set**: Ensure `GOOGLE_CLOUD_PROJECT_ID` environment variable is set
5. **Bucket already exists**: The test bucket name might already be in use globally
6. **File not found**: Ensure test files (`env.example`, `gcs_client.py`) exist in the project root
7. **Permission denied**: Check that your service account has read/write access to GCS
8. **Import errors**: Make sure you're running tests from the project root directory with venv activated

### Debug Mode

Run tests with increased verbosity to see more details:

```bash
python -m pytest tests/integration_test.py -v -s
```

The `-s` flag allows print statements to be displayed, which can help with debugging.

## Test Data

The tests use these files:

- **Local files**: `env.example`, `gcs_client.py` (must exist in project root)
- **Test bucket**: `test-bucket-integration` (created and deleted during tests)
- **Test files**: Same names as local files, uploaded to the test bucket

## Cleanup

### Test Cleanup

The tests are designed to clean up after themselves by deleting the test bucket at the end. If tests fail partway through, you may need to manually delete the `test-bucket-integration` bucket from your GCS console.

### Virtual Environment Cleanup

When you're done testing, you can deactivate the virtual environment:

```bash
deactivate
```

To remove the virtual environment entirely:

```bash
# Deactivate first
deactivate

# Remove the venv directory
rm -rf venv
```
