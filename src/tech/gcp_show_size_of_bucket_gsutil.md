# Google Cloud Platform: Show Size of Bucket Using gsutil

```sh
gsutil du -s gs://BUCKET_NAME
```

This returns the size of the bucket in bytes.  Don't run this if you have hundreds of thousands of objects in the bucket, as it makes individual requests and can take a long time.

[See the official doc.](https://cloud.google.com/storage/docs/getting-bucket-information#gsutil)