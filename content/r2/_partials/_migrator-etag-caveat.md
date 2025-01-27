---
_build:
  publishResources: false
  render: never
  list: never
inputParameters: param1
---

While R2's ETag generation is compatible with S3's during the regular course of operations, ETags are not guaranteed to be equal when an object is migrated using $1.
$1 makes autonomous decisions about the operations it uses when migrating objects to optimize for performance and network usage. It may choose to migrate an object in multiple parts, which affects [ETag calculation](/r2/objects/multipart-objects#etags).

For example, a 320 MiB object originally uploaded to S3 using a single `PutObject` operation might be migrated to R2 via multipart operations. In this case, its ETag on R2 will not be the same as its ETag on S3.
Similarly, an object originally uploaded to S3 using multipart operations might also have a different ETag on R2 if the part sizes $1 chooses for its migration differ from the part sizes this object was originally uploaded with.

Relying on matching ETags before and after the migration is therefore discouraged.
