s3wipe
======

A rapid parallelized AWS S3 key &amp; bucket deleter.

I was recently tasked with deleting an bucket in Amazon's Simple Storage 
Service (S3) that contained an absolutely massive number of files.

Unfortunately, Amazon themselves do not give you an easy way to do this 
yourself.  Their web interface stalls indefinitely when you delete an
"adequately large" number of files, and their CLI tool (aptly named 
"aws-cli") only deletes files in a single-threaded fashion (i.e. slowly).

After googling around a bit, I encountered 
[s3nukem](https://github.com/lathanh/s3nukem) (itself a fork
of [s3nuke](http://github.com/SFEley/s3nuke/)), which appeared to be the
solution to my problems.  After a few minutes of trying to find the 
'right' version of RightAWS (the s3nukem code & readme had a
disagreement over this), I was able to get it up and running.  However,
after a bit of back-of-the-napkin math, it was looking like it was still
going to take at least a month of running s3nukem before the bucket was 
deleted. 

So, I wrote s3wipe.  S3wipe, as far as I know, is the only S3 key/bucket 
deletion tool that:

* Does parallel, thread-based delete AND list operations (more speed)
* Performs batch deletes (MOAR SPEED!)
* Will delete versioned objects (MOAR... well, deletes)

Using s3wipe, I was able to delete _400 million S3 objects in about 24 hours_.

## Installation

This is just a single-file script, so just go ahead and run it.  It will need
a semi-recent version of the "boto" python module to be installed, though, so:

    pip install boto

_or_

    yum install python-boto

_or_

    apt-get install python-boto

Then:

    wget https://raw.github.com/slmingol/s3wipe/master/s3wipe
    chmod 755 s3wipe

### Using Docker

Clone the repo:

    git clone git@github.com:slmingol/s3wipe.git
    cd s3wipe

Build the Docker image:

    docker build -t s3wipe:latest .

Then run the script:

    docker run s3wipe:latest --help

## Usage

```
usage: s3wipe [-h] --path PATH --id ID --key KEY [--dryrun] [--quiet]
              [--batchsize BATCHSIZE] [--maxqueue MAXQUEUE] [--delbucket]

Recursively delete all keys in an S3 path

optional arguments:
  -h, --help             show this help message and exit
  --path PATH            S3 path to delete (e.g. s3://bucket/path)
  --id ID                Your AWS access key ID
  --key KEY              Your AWS secret access key
  --dryrun               Don't delete. Print what we would have deleted
  --quiet                Suprress all non-error output
  --batchsize BATCHSIZE  # of keys to batch delete (default 100)
  --maxqueue MAXQUEUE    Max size of deletion queue (default 10k)
  --delbucket            If S3 path is a bucket path, delete the bucket also
  --region               Region of target S3 bucket. Default vaue `us-east-1`

```

## Example run

**NOTE:** The below will automatically pull the container from Github prior to executing it if you don't already have it.

```
$ docker run -it --rm slmingol/s3wipe \
   --id $(aws configure get default.aws_access_key_id) \
   --key $(aws configure get default.aws_secret_access_key) \
   --path s3://bw-tf-backends-aws-example-logs \
   --delbucket
[2019-02-20@03:58:33] INFO: Deleting from bucket: bw-tf-backends-aws-example-logs, path: None
[2019-02-20@03:58:33] INFO: Getting subdirs to feed to list threads
[2019-02-20@03:58:35] INFO: Starting 2 delete threads...
[2019-02-20@03:58:35] INFO: Starting 1 list threads...
[2019-02-20@03:58:37] INFO: Done deleting keys
[2019-02-20@03:58:37] INFO: Bucket is empty.  Attempting to remove bucket
```

## Changelog

_v0.3_

    Put everything together so that it's on docker hub and linked to the src
    on github.

_v0.2_

    You can now delete all keys under an arbitrary S3 path, instead of only
    entire buckets (although that is still an option, as well).

_v0.1_

    Initial version.
    
## References
- https://cloud.docker.com/u/slmingol/repository/docker/slmingol/s3wipe
- [How do I delete a versioned bucket in AWS S3 using the CLI?
](https://stackoverflow.com/questions/29809105/how-do-i-delete-a-versioned-bucket-in-aws-s3-using-the-cli/54778540#54778540)
