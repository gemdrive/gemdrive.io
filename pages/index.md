# What is GemDrive?

GemDrive is a protocol and [reference implementation][0] for bringing
filesystem-like functionality to web browsers and other HTTP clients. It shares
similarities with WebDAV, NFS, FTP, Inrupt Solid, remoteStorage, Amazon S3 (the
protocol), etc. For detailed comparisons with other tools, see [this page][1]


# Motivation

The goal of GemDrive is to provide the "missing hard drive for the web". That
means providing a way for web apps to read and write to a server in a similar
way as a normal filesystem. If you've ever used an app that supports using
Google Drive to store your data, it's the same idea, just much simpler and open
source.

GemDrive seeks to do this by adding the minimal necessary layer on top of
standard HTTP.

HTTP already provides most of the necessary functionality to do this. The
following HTTP request types already behave in fairly intuitive ways for
filesystem operations:

* HTTP GET for reading files, including partial reads (via byte-range
  requests).
* HTTP PUT for uploading files and creating directories.
* HTTP DELETE for deleting files and directories.

However, a few pieces are still missing/underspecified:

* Listing directory contents.
* Partial writes (update only a portion of a file).
* Authentication and authorization.
* Copying between remote servers without downloading the data to the client.

Other tools in this space provide solutions to the above. The following
features are what make GemDrive unique:

* Extremely simple. You can create a useful GemDrive client or server in a few
  lines of code, with no libraries required. A central goal is for the protocol
  to be simple enough that if you were going to implement HTTP filesystem-like
  functionality yourself (as [many projects][3] already have), you may as well
  implement GemDrive.
* The protocol supports rich functionality, but is intended to be implemented
  incrementally. For example, if you only need public reads, there's no need to
  implement writes or auth to still be considered compliant.
* Designed to combine well with other tools. For example, you could easily
  add GemDrive compatibility to a Solid server, or nginx/Caddy/etc. You can
  also turn any static web server into a (read-only) GemDrive server by
  enabling CORS and generating a static index.
* Hosting web pages and static apps from a GemDrive server is a first-class
  use case. For example, most of my websites (including this one) are served
  from my personal GemDrive server.
* Specified support for basic image resizing. This is crucial for things like
  remote file explorers to generate thumbnails and previews.
* It's easy to run your own GemDrive server on your local machine and access it
  over localhost. This provides a way for apps to directly (and securely)
  access your local filesystem. This opens up a wide range of applications such
  as browser-based video editing of local files, music players, sync with
  remote GemDrive servers, etc.


# What does it look like?

Here's a taste of GemDrive in action. For a complete description, see the
[protocol page][2].

Note that for brevity the requests below don't show the necessary auth. When
the server starts up it prints a master key which then must be included either
in the Authorization: Bearer header or the access_token query parameter.

Alternatively you can mark a directory or file as public. We'll assume that has
already been done for the following requests.

## Start a local server

```bash
./gemdrive-server -dir dir1 -dir dir2
```

## List the root directory

```bash
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

```bash
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

```bash
curl localhost:3838/dir1/f1.txt

```

```
Hi there
```

## Read part of a file

```bash
curl -H "Range: bytes=0-3" localhost:3838/dir1/f1.txt

```

```
Hi t
```

## Upload a file

```bash
curl -X PUT localhost:3838/dir1/f1.txt?overwrite=true -d 'New text'
curl localhost:3838/dir1/f1.txt

```
```
New text
```

A few things to note:

* GemDrive-specific requests start with /gemdrive/.
* Non-GemDrive requests are the name as a vanilla webserver (except they must
  return CORS headers).
* Requests to the GemDrive index (ie directory listings) start with
  /gemdrive/index/ and end with list.json, with the directory path in between.
* Index responses are returned as simple json.
* File size and modification times are included (to facilitate syncing tools,
  for example).
* Directory names end in "/", files do not.
* The protocol is set up in such a way that you can pre-generate the json files
  for the index and serve them as static files alongside your data. So you
  could host a (read-only) GemDrive instance on a public S3 bucket, for
  example.

[0]: https://github.com/gemdrive/gemdrive-go

[1]: /comparisons/

[2]: /protocol/

[3]: https://github.com/awesome-selfhosted/awesome-selfhosted#file-transfer---web-based-file-managers
