# DVC Push to Google Cloud Storage — Troubleshooting Log

## Problem

`dvc push` to a GCS remote (`gs://bonga-bucket/dvc-storage`) failed with two successive errors.

### Error 1: Missing Application Default Credentials

```
Could not determine bucket type for bucket name bonga-bucket:
Your default credentials were not found. To set up Application Default Credentials,
see https://cloud.google.com/docs/authentication/external/set-up-adc for more information.
```

### Error 2: SSL Certificate Verification Failure

After fixing ADC, `dvc push` failed again with:

```
SSLCertVerificationError: [SSL: CERTIFICATE_VERIFY_FAILED] certificate verify failed:
unable to get local issuer certificate (_ssl.c:1032)

Cannot connect to host storage.googleapis.com:443 ssl:True
```

Full traceback went through `gcsfs` → `aiohttp` → `asyncio` → `ssl`.

## Root Causes

1. **No Application Default Credentials (ADC).** DVC uses `gcsfs`, which authenticates via ADC — not via `gcloud auth login` user credentials. Without ADC, gcsfs could not determine the bucket type or authenticate.

2. **Python could not find root CA certificates.** Python 3.13 was installed from python.org (located at `/Library/Frameworks/Python.framework/Versions/3.13/`). The python.org installer on macOS does not link Python's `ssl` module to the system's root CA certificates. As a result, any HTTPS connection from Python (including gcsfs/aiohttp to `storage.googleapis.com`) failed TLS verification with `CERTIFICATE_VERIFY_FAILED`.

## Fix

### Step 1 — Set up Application Default Credentials

```bash
gcloud auth application-default login
```

This opened a browser, authenticated as `kevin.bonga@gmail.com`, and saved credentials to:

```
/Users/kevin/.config/gcloud/application_default_credentials.json
```

Verified with:

```bash
gcloud auth application-default print-access-token
# → returned a valid ya29.... token
```

### Step 2 — Install CA certificates for Python

Ran the script that ships with the python.org macOS installer:

```bash
"/Applications/Python 3.13/Install Certificates.command"
```

Output:

```
 -- pip install --upgrade certifi
 -- removing any existing file or link
 -- creating symlink to certifi certificate bundle
 -- setting permissions
 -- update complete
```

This symlinks the `certifi` CA bundle into Python's OpenSSL directory:

```
/Library/Frameworks/Python.framework/Versions/3.13/etc/openssl/cert.pem
```

### Step 3 — Verify the fix

Tested GCS connectivity from Python:

```bash
.venv/bin/python -c "
import gcsfs
fs = gcsfs.GCSFileSystem(token='google_default')
print(fs.ls('gs://bonga-bucket'))
"
# → []  (bucket accessible, empty)
```

### Step 4 — Push with DVC

```bash
.venv/bin/dvc push
```

Result:

```
1516 files pushed
```

## What Was Actually Performed

| Step | Command | Purpose |
|------|---------|---------|
| 1 | `gcloud auth application-default login` | Created ADC so gcsfs/DVC can authenticate to GCS |
| 2 | `gcloud auth application-default print-access-token` | Verified ADC is working |
| 3 | `"/Applications/Python 3.13/Install Certificates.command"` | Linked certifi CA bundle into Python's OpenSSL trust store |
| 4 | `.venv/bin/python -c "...gcsfs test..."` | Verified Python can now make HTTPS calls to GCS |
| 5 | `.venv/bin/dvc push` | Pushed 1516 files to `gs://bonga-bucket/dvc-storage` |

## Notes

- If Python is reinstalled or upgraded, the **Install Certificates.command** must be run again.
- The DVC remote was already configured correctly:
  ```
  myremote  gs://bonga-bucket/dvc-storage  (default)
  ```
- `gsutil ls` worked throughout because it uses gcloud's own credential and TLS stack, which is why the bucket was reachable via gcloud but not via DVC/gcsfs.
