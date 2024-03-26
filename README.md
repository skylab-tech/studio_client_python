# Skylab Studio Python Client

SkylabTech Studio Python client.

[studio.skylabtech.ai](https://studio.skylabtech.ai)

## Requirements

libvips is required to be installed on your machine in order to install skylab-studio (for pyvips).

- [Libvips documentation](https://www.libvips.org/install.html)

pyvips v2.2.1 is required. If you encounter trouble installing pyvips == 2.2.1, for the time being you can use a conda environment and install pyvips there.

```bash
$ conda install --channel conda-forge pyvips
```

## Installation

```bash
$ pip install skylab-studio
```

## Example usage

```python
import skylab_studio

# CREATE PROFILE
payload = {
  "name": "profile name",
}

api.create_profile(payload=payload)

# CREATE JOB
payload={
  "name": "job name",
  "profile_id": profile_id
}

job = api.create_job(payload)

# UPLOAD JOB PHOTO(S)
filePath = "/path/to/photo"
api.upload_job_photo(filePath, job.id)

# QUEUE JOB
payload = { "callback_url" = "YOUR_CALLBACK_ENDPOINT" }
api.queue_job(job.id, payload)

# NOTE: Once the job is queued, it will get processed then complete
# We will send a response to the specified callback_url with the output photo download urls
```

```python
# OPTIONAL: If you want this SDK to handle photo downloads to a specified output folder

# FETCH COMPLETED JOB (wait until job status is completed)
completed_job = api.get_job(queued_job['id']);

# DOWNLOAD COMPLETED JOB PHOTOS
photos_list = completed_job['photos'];
await api.download_all_photos(photos_list, completed_job['profile'], "photos/output/");
```

## Usage

For all examples, assume:

```python
import skylab_studio

api = skylab_studio.api(api_key='YOUR-API-KEY')
```

### Error Handling

By default, the API calls return a response object no matter the type of response.

### Endpoints

#### List all jobs

```python
api.list_jobs()
```

#### Create job

```python
payload = {
  'profile_id': 1
}

api.create_job(payload=payload)
```

For all payload options, consult the [API documentation](https://studio-docs.skylabtech.ai/#tag/job/operation/createJob).

#### Get job

```python
api.get_job(job_id)
```

#### Update job

```python
payload = {
  'profile_id': 2
}

api.update_job(job_id, payload=payload)
```

For all payload options, consult the [API documentation](https://studio-docs.skylabtech.ai/#tag/job/operation/updateJobById).

#### Queue job

```python
payload = {
  "callback_url": "desired_callback_url"
}

api.queue_job(job_id, payload)
```

#### Delete job

```python
api.delete_job(job_id)
```

#### Cancel job

```python
api.cancel_job(job_id)
```

#### Jobs in front

Use after queueing job to check number of jobs ahead of yours

```python
api.fetch_jobs_in_front(job_id)
```

#### List all profiles

```python
api.list_profiles()
```

#### Create profile

```python
payload = {
  'name': 'My profile'
}

api.create_profile(payload=payload)
```

For all payload options, consult the [API documentation](https://studio-docs.skylabtech.ai/#tag/profile/operation/createProfile).

#### Get profile

```python
api.get_profile(profile_id)
```

#### Update profile

```python
payload = {
  'name': 'My profile'
}

api.update_profile(profile_id, payload=payload)
```

For all payload options, consult the [API documentation](https://studio-docs.skylabtech.ai/#tag/profile/operation/updateProfileById).

#### Get photo

```python
api.get_photo(photo_id)
```

#### Upload job photo

This function handles validating a photo, creating a photo object and uploading it to your job/profile's s3 bucket. If the bucket upload process fails, it retries 3 times and if failures persist, the photo object is deleted.

```python
api.upload_job_photo(photo_path, job_id)
```

#### Upload profile photo

This function handles validating a background photo for a profile. Note: enable_extract and replace_background (profile attributes) MUST be true in order to create background photos. Follows the same upload process as upload_job_photo.

```python
api.upload_profile_photo(photo_path, profile_id)
```

`Returns: { photo: { photo_object }, upload_response: bucket_upload_response_status }`

If upload fails, the photo object is deleted for you. If upload succeeds and you later decide you no longer want to include that image, use delete_photo to remove it.

#### Download photo(s)

This function handles downloading the output photos to a specified directory.

```python
photos_list = completed_job.photos;

download_results = await api.download_all_photos(photos_list, completed_job.profile, "/output/folder/path");

print(download_results)

Output:
{'success_photos': ['1.JPG'], 'errored_photos': []}
```

OR

```python
api.download_photo(photo_id, "/output/folder/path");
```

#### Delete photo

This will remove the photo from the job/profile's bucket. Useful for when you've accidentally uploaded an image that you'd like removed.

```python
api.delete_photo(photo_id)
```

#### Validate hmac headers

Applicable if you utilize the job callback url. Use to validate the job payload integrity.

- secret_key (string): Obtain from Skylab

- job_json (string): Stringified json object obtained from callback PATCH request

- request_timestamp (string): Obtained from callback PATCH request header 'X-Skylab-Timestamp'

- signature (string): Signature generated by Skylab to compare. Obtained from callback PATCH request header 'X-Skylab-Signature'

Returns **True** or **False** based on whether or not the signatures match.

```python
api.validate_hmac_headers(secret_key, job_json, request_timestamp, signature)
```

### Expected Responses

#### Success

```bash
    >>> response.status_code
    200

    >>> response.json().get('success')
    True

    >>> response.json().get('status')
    u'OK'

    >>> response.json().get('profile_id')
    u'numeric-profile-id'
```

#### Error

- Malformed request

```bash
    >>> response.status_code
    422
```

- Bad API key

```bash
    >>> response.status_code
    403
```

## Troubleshooting

### General Troubleshooting

- Enable debug mode
- Make sure you're using the latest Python client
- Capture the response data and check your logs &mdash; often this will have the exact error

### Enable Debug Mode

Debug mode prints out the underlying request information as well as the data
payload that gets sent to Studio. You will most likely find this information
in your logs. To enable it, simply put `debug=True` as a parameter when instantiating
the API object. Use the debug mode to compare the data payload getting
sent to [Studio' API docs](http://docs.studio.skylabtech.ai/#).

```python
import skylab_studio

api = skylab_studio.api(api_key='YOUR-API-KEY', debug=True)
```

### Response Ranges

Studio' API typically sends responses back in these ranges:

- 2xx – Successful Request
- 4xx – Failed Request (Client error)
- 5xx – Failed Request (Server error)

If you're receiving an error in the 400 response range follow these steps:

- Double check the data and ID's getting passed to Studio
- Ensure your API key is correct
- Log and check the body of the response

## Distribution

To package:

```bash
  python -m build
  python -m twine upload dist/*
```
