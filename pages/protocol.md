# Core concepts

The GemDrive protocol is designed with the following guiding principles:

* As simple as reasonable. What are the ~20% of features that will enable ~80%
  of use cases?
* Can be implemented partially/incrementally. If your implementation doesn't
  need specific features, they can usually be left out. In practice this means
  a GemDrive client should fail gracefully, as a server may return a 4xx for
  nearly any request.
* The GemDrive requests are designed in such a way that they can be represented
  as simple files on disk. This means that the GemDrive index and downsized
  images can be pre-generated and served by a static file server, turning
  almost any webserver into a compatible GemDrive server.


The various request types are order below, roughly in the order of importance,
ie the order they would be implemented in.


# List file metadata

**`HEAD /<path>`**

File list operations are supported using vanilla HTTP HEAD requests. Currently
only `Content-Length` is required to be returned by the server. A custom header
for `modTime` may be required in the future.

Examples:

```plaintext
curl --head localhost:3838/path/to/file.txt
```


# Read a file

**`GET /<path>`**

File reads are the most basic type of request. These are supported as vanilla
HTTP GET requests. Support for HTTP GET is required. Support for HTTP Range
requests is highly encourage, and may be required in future protocol revisions.

Technically, a static webserver is a valid GemDrive server. In practice, it
should set CORS headers in order to be useful to web apps.

Examples:

```plaintext
curl localhost:3838/path/to/file.txt
```


# List a single directory

**`GET /gemdrive/index/<path>/list.json`**

This request must return a JSON object of the form:

```json
{
  "children": {
    "file1.txt": {
      "size": 9,
      "modTime": "2021-08-18T03:10:16Z"
    },
    "directory1/": {
      "size": 4096,
      "modTime": "2021-08-18T03:09:59Z"
    }
  }
}
```

Where:

* Directory item names end in "/", file item names do not.
* `size` is an integer.
* `modTime` is an ISO 8601 string.

## Examples:

```plaintext
curl localhost:3838/gemdrive/index/path/to/directory/list.json
```

# Create a key

**`POST /gemdrive/create-key`**

Most of the remaining requests should nearly always be protected against public
access, as they can modify the state of the server. The GemDrive protocol
specifies an extremely simple capability-based security system for
implementing authorized requests. This system revolves around the concept of
keys. Keys exist in a tree. The root key must be retrieved by the system
administrator when the server is first started. Any key can be used to create
other keys which have equal-or-lesser privileges from the parent key. If a key
is deleted, all descendant keys must also be removed from the system.

Keys are included in requests either by setting the `access_token` query
parameter, or the `Authorization: Bearer` header.

## Examples:

Assume you have a `key_request.json` file with the following contents:

```json
{
  "privileges": {
      "/dir/": "read"
  }
}
```

And a pre-existing key with the value `dHaa6SZx72rR2vqJDmpHcvxTQJHRre91` which
maps to the following (root in this case) privileges:

```json
{
  "privileges": {
      "/": "write"
  }
}
```

The following request will return a key that has permissions to read `/dir/`.

```plaintext
curl -X POST -T @key_request.json localhost:3838/gemdrive/create-key?access_token=dHaa6SZx72rR2vqJDmpHcvxTQJHRre91
```


# Create a file

**`PUT /<path>`**

File creation is supported by vanilla HTTP PUT requests. `<path>` must not
end in "/".

Final size on disk on
the server will match the `Content-Length` header of the request. Empty files
can be created by sending a 0-sized file.

## Parameters 

* **`overwrite (default false)`** - Overwrite file if it already exists. If
  `false` and file exists, server should return 4xx and not modify the file.

## Examples:

```plaintext
curl -X PUT -d "Hi there" localhost:3838/path/to/file.txt?overwrite=true
```


# Create a directory

**`PUT /<path>`**

Directory creation is supported by vanilla HTTP PUT requests. The `<path>`
parameter must end in "/".


## Parameters 

* **`recursive (default false)`** - Indicates whether to create parent
  directories. This is analogous to `mkdir -p`. If `false` and one or more
  parent directories don't already exists, server should return a 4xx status
  and not modify the filesystem.


## Examples:

```plaintext
curl -X PUT localhost:3838/path/to/directory/?recursive=true
```

# Delete a file

**`DELETE /<path>`**

File deletion is supported by vanilla HTTP DELETE requests. The `<path>`
parameter must not end in "/".

## Examples:

```plaintext
curl -X DELETE localhost:3838/path/to/file.txt
```


# Delete a directory

**`DELETE /<path>`**

Directory deletion is supported by vanilla HTTP DELETE requests. The `<path>`
parameter must end in "/".

## Parameters 

* **`recursive (default false)`** - Indicates whether to delete non-empty
  directories. If `false` and directory is not empty, server should return a
  4xx error.

## Examples:

```plaintext
curl -X DELETE localhost:3838/path/to/directory/?recursive=true
```



# Modify a file (partial writes and appends)

**`PATCH /<path>`**

Partial writes are supported by HTTP PATCH requests. `<path>` must not end in
"/". If a server does not support partial writes (ie object storage backends),
it should return HTTP 405.

The `Content-Length` request header is used to determine the number of bytes
written.

## Parameters 

* **`offset (default 0)`** - Offset in bytes from which to start writing. For
  append operations, this should match the current size of the file on disk as
  returned by a HEAD request.

## Examples:

```plaintext
curl -X PATCH localhost:3838/path/to/file.txt?offset=10
```


# Retrieve a downsized file

**`/gemdrive/images/<pixel_size>/<path>`**

GemDrive specifices basic support for the server to provide downsized versions
of image files. While this may seem a strange feature to include in a barebones
protocol, in practice this functionality is essential for any sort of useful
file explorer or gallery apps. The goal is to support thumbnail, preview, and
fullscreen image sizes. Currently only 3 sizes are required to support this
feature: `128x128` (thumbnail), `512x512` (preview), and `2048x2048`
(fullscreen). These values are likely to change or be added to in the future.
Unfortunately supporting arbitrary sizes is not possible because it opens up a
DOS vulnerability whereby an attacker can send a bunch of requests asking for
every possible combination of sizes.

In the request path, `<pixel_size>` represents the largest dimension of the
image. The server must return an image that is equal to that number in either
width or height, while preserving the original aspect ratio, and not being
larger than that number in the other dimension.

## Examples:

Assume your GemDrive server has an image hosted at `/dir/image.jpeg`.

Retrieve a thumbnail of the image:

```plaintext
curl localhost:3838/gemdrive/images/128/dir/image.jpeg
```

Retrieve a preview of the image:

```plaintext
curl localhost:3838/gemdrive/images/512/dir/image.jpeg
```

Retrieve a fullscreen-sized copy of the image:

```plaintext
curl localhost:3838/gemdrive/images/2048/dir/image.jpeg
```


# List a directory tree

**`/gemdrive/index/<path>/tree.json`**

This request returns a JSON object in the same format as `list.json`, except it
may be nested to arbitrary depth:

```json
{
  "children": {
    "file1.txt": {
      "size": 9,
      "modTime": "2021-08-18T03:10:16Z"
    },
    "directory1/": {
      "size": 4096,
      "modTime": "2021-08-18T03:09:59Z",
      "children": {
         "file2.txt": {
           "size": 29,
           "modTime": "2021-08-19T03:20:06Z"
         }
      }
    }
  }
}
```

It is particularly useful for operations like directory synchronization, and
can greatly improve performance for deeply nested trees by reducing the number
of requests necessary.


## Parameters 

* **`depth (default 0)`** - Hint for tree depth, ie how deep of a directory tree to return.
  Servers are not required to honor this parameter, and may return a tree that
  is either deeper or more shallow than requested. Default is 0, which means
  return the entire tree.

## Examples:

```plaintext
curl localhost:3838/gemdrive/index/path/to/directory/tree.json?depth=2
```



