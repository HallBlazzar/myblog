+++
title = "Watchout Cloud Run With Cloud Storage Volumes"
date = 2026-04-13
draft = false
categories = ["GCP"]
+++

Few things I recently encountered while deploying Cloud Run services with Cloud Storage volumes on GCP, which I think worth of sharing.

## Container Exit Without Reporting Errors

In general, when creating a Cloud Run service with Cloud Storage volume, the Service Account used to perform the service creation operation should have Storage Object User or Admin role. If not, you might see:

1. Keeping seeing container exit related errors from Log Explorer:

    ```text
    "textPayload": "Application exec likely failed"
    "textPayload": "terminated: Application failed to start: The container may have exited abnormally."
    ```

2. Cloud Storage volumes looks mounted successfully from Log Explorer:

    ```text
    "textPayload": "time="DD/mm/YYYY HH:MM:SS.ffffff" severity=INFO message="Mounting file system \"some-filesystem\"..." mount-id=some-filesystem-xxxxxxxx"
    ```

3. No matter the way users adjust entrypoint and adding logs, containers just exit as 1. without logs users expected.

Be aware that unlike Azure Container Apps or AWS ECS, creating services with network filesystems without proper permissions on GCP Cloud Run won't fail operation immediately. On contrary, users still see filesystem mounted successfully as usual but cannot run container endpoints.

## File Access Issue On Cloud Storage Volume

Beware of the error message like:

```text
"textPayload": "time="DD/mm/YYYY HH:MM:SS.ffffff" severity=ERROR message="BufferedWriteHandler.OutOfOrderError for object: db.sqlite-shm, expectedOffset: 0, actualOffset: 4095" mount-id=some-filesystem-xxxxxxxx"
"textPayload": "time="DD/mm/YYYY HH:MM:SS.ffffff" severity=INFO message="Out of order write detected. File db.sqlite-shm will now use legacy staged writes. Streaming writes is supported for sequential writes to new/empty files. For more details, see: https://github.com/GoogleCloudPlatform/gcsfuse/blob/master/docs/semantics.md#writes" mount-id=some-filesystem-xxxxxxxx"
```

In my use case, I tried to create a SQLite database on Cloud Storage volume and used the [better-sqlite3](https://github.com/WiseLibs/better-sqlite3) to access it. The result was, data was written to copies of containers' memory but to the database file on Cloud Storage. Also observe the error messages above in **INFO** log level on Log Explorer.

Though I tried to follow [the link](https://github.com/GoogleCloudPlatform/gcsfuse/blob/master/docs/semantics.md#writes) in the error message, but I didn't find information can explain the situation. There is a [Reddit topic](https://www.reddit.com/r/googlecloud/comments/1afm74h/sqlite_running_in_cloud_run_with_gcs_volume_mount/) discussed this approach and succeeded (with high latency), but the original poster used Python Flask and SQLite3 library, which might work differently from NodeJs. 

My assumption is, object store services might have issues with partially write/stream related operation. The operation to files in object store services are designed to apply to **files** instead of **their content**. There are some [posts](https://www.reddit.com/r/aws/comments/dplfoa/why_is_s3fs_such_a_bad_idea/) people discussed problems could be brought by using object store services as filesystems.

My best advice of this issue is, unless we can ensure file operations on Cloud Storage volumes are file level operations, otherwise, EFS should be safer. In my use case, I still rewrite the project to use PostgreSQL and Cloud SQL instead.
