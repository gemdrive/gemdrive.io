# What is GemDrive?

GemDrive is a [protocol](./protocol/) and [reference implementation][0] for
bringing remote filesystem functionality to web browsers and other HTTP
clients.  It shares similarities with WebDAV, NFS, FTP, Inrupt Solid,
remoteStorage, Amazon S3, Google Drive, etc. For detailed comparisons with
other tools, see [this page][1].

GemDrive grew out of a combination of needs I had for managing my own data, as
well as needs I ran into at my day job working with large genomic datasets.

It is currently targeted at developers and self-hosters, but will hopefully be
useful to a wider audience in the long run.


# Why does it exist?

The goal of GemDrive is to provide a "hard drive" for the web. That means
providing a way for web apps to read and write to a server in a similar way as
a normal filesystem (files and folders), rather than the database query
paradigm normally used for the web.  If you've ever used an app that supports
using Google Drive to store your data, GemDrive is the same idea, but much
simpler, open source, and providing extra features that enable some really cool
stuff.

To understand some of the motivations behind GemDrive, consider how you would
accomplish the following with tools available today:

* Share a cloud storage folder with someone (including write permissions)
  without requiring them to have an account with the provider.
* Host a website from your cloud storage.
  * Major cloud storage providers used to support this, but I'm not aware of
    any that still do (see [here][6] and [here][7]).
* Write a web app to access a user's cloud storage.
  * Google Drive has been the gold standard for this in the past, but it
    requires complicated auth flows and libraries, and recent versions are
    riddled with [confusing limitations][5].
* Host your own cloud storage server, while still being able to directly
  transfer data from your server to someone else's server without first
  downloading the data locally.
* Publicly share a large folder that can be recursively/partially downloaded.
  * rsync daemon doesn't encrypt the data in transit.
  * rsync over SSH depends on sharing SSH keys or creating user accounts.
  * rsync requires rsync.
  * AWS CLI requires credentials, which usually means an AWS account.
  * Zipping is time consuming, uses extra storage for the zipped file, and
    makes it difficult to update only a portion of the data.
* Run a [static site generator][8] for your website entirely within your web
  browser.
* Write a web app capable of accessing a user's
  local hard drive.
  * The [File System Access API][4] isn't ready yet.

I'm certain there are nice solutions to some of these problems (and would
appreciate people letting me know about them). GemDrive is pretty good at
solving all of them.


# How is it implemented?

**The** central tenet of GemDrive is to be as simple as reasonable.

The protocol seeks to add the minimal necessary layer on top of standard HTTP
to do its job.

The following HTTP request types already behave in fairly intuitive ways for
filesystem operations:

* HTTP GET for reading files, including partial reads (via byte-range
  requests).
* HTTP PUT for creating/uploading files and creating directories.
* HTTP DELETE for deleting files and directories.

However, a few pieces are still missing/underspecified:

* Listing directory contents. GemDrive would be useful if it *only* added this.
* Partial writes (update only a portion of a file).
* Authentication and authorization.
* Copying between remote servers without downloading the data to the client.
* Protections against accidentally deleting large amounts of data.

GemDrive simply provides opinionated answers for these pieces.


# What does it look like?

Here's a taste of GemDrive in action. For a complete description, see the
[protocol page][2].

Note that for brevity the requests below don't show the usual auth. When the
server starts up it prints a master key which then must be included either in
the `Authorization: Bearer <token>` header or the `access_token` query
parameter.

Alternatively you can mark a directory or file as public. We'll assume that has
already been done for the following requests, although it's something you
would pretty much never want to do even on localhost.


## Preliminaries

Before we run the examples, first create some dummy data to work on:

```plaintext
mkdir -p dir1/subdir1
echo Hi there > dir1/f1.txt
mkdir dir2
echo Hello there > dir2/f2.txt
```

You should now have the following directory structure on disk:

```plaintext
dir1/
    f1.txt
    subdir1/
dir2/
    f2.txt
```

## Start a local server

```plaintext
./gemdrive-server -dir dir1 -dir dir2
```

## List the root directory

```plaintext
curl localhost:3838/gemdrive/index/list.json

```

```json
{
  "children": {
    "dir1/": {},
    "dir2/": {}
  }
}
```

## List a subdirectory

```plaintext
curl localhost:3838/gemdrive/index/dir1/list.json

```

```json
{
  "children": {
    "f1.txt": {
      "size": 9,
      "modTime": "2021-08-18T03:10:16Z"
    },
    "subdir1/": {
      "size": 4096,
      "modTime": "2021-08-18T03:09:59Z"
    }
  }
}
```


## Read a file

```plaintext
curl localhost:3838/dir1/f1.txt

```

```plaintext
Hi there
```

## Read part of a file

```plaintext
curl -H "Range: bytes=0-3" localhost:3838/dir1/f1.txt

```

```plaintext
Hi t
```

## Upload a file

```plaintext
curl -X PUT localhost:3838/dir1/f1.txt?overwrite=true -d 'New text'
curl localhost:3838/dir1/f1.txt

```
```plaintext
New text
```

A few things to note:

* GemDrive-specific requests start with `/gemdrive/`.
* Non-GemDrive requests are the name as a vanilla webserver.
* Requests to the GemDrive index (ie directory listings) start with
  `/gemdrive/index/` and end with `list.json`, with the directory path in between.
* Index responses are returned as simple JSON.
* File size and modification times are included (to facilitate syncing tools,
  for example).
* Directory names end in "/", files do not.
* The protocol is set up in such a way that you can pre-generate the JSON files
  for the index and serve them as static files alongside your data. So you
  could host a (read-only) GemDrive instance on a public S3 bucket (or any
  other webserver).

[0]: https://github.com/gemdrive/gemdrive-go

[1]: /comparisons/

[2]: /protocol/

[3]: https://github.com/awesome-selfhosted/awesome-selfhosted#file-transfer---web-based-file-managers

[4]: https://developer.mozilla.org/en-US/docs/Web/API/File_System_Access_API

[5]: https://gdrivemusic.com/help

[6]: https://help.dropbox.com/files-folders/share/public-folder

[7]: https://workspaceupdates.googleblog.com/2015/08/deprecating-web-hosting-support-in.html

[8]: https://jamstack.org/generators/

[9]: https://github.com/gemdrive/gemdrive-ro-server-js/blob/master/index.js
