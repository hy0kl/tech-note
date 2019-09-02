# aws s3, cp or sync?

```
Using aws s3 cp from the AWS Command-Line Interface (CLI) will require the --recursive parameter to copy multiple files.
aws s3 cp s3://myBucket/dir localdir --recursive

The aws s3 sync command will, by default, copy a whole directory. It will only copy new/modified files.
aws s3 sync s3://mybucket/dir localdir
```
