# Solid

I'm bullish on [Solid][1], and I think GemDrive compliments it nicely. While
GemDrive aims to provide the minimal necessary filesystem features, Solid
takes a comprehensive top-down approach to providing a personal (and private)
storage solution.

The main advantage GemDrive has over Solid is simplicity. Solid requires
compliance with various different standards. The `package.json` file for the
community server lists over 50 dependencies. GemDrive is just a minimal JSON
layer on HTTP. You don't need client libraries to talk to a server. You don't
even need libraries to *write* [a server][2]. The [reference server][3] ships
as a single static executable. It also currently has 3 dependencies, one of
which I wrote primarily for GemDrive.


# remoteStorage

[remoteStorage][0] is an excellent protocol that is very similar to GemDrive. The
differences are mostly in the details.


* GemDrive can be hosted entirely from a static fileserver, by pre-generating
  the index.
* GemDrive supports server-to-server transfers.
* GemDrive supports modifying files/partial uploads.
* GemDrive specifies a way for the server to provide thumbnails and previews,
  which is critical for remotely browsing data.
* GemDrive supports hosting web pages as a first-class use case.
* remoteStorage used JSON-LD. GemDrive is simple JSON.
* remoteStorage uses WebFinger.


# WebDAV

WebDAV is so close to being what we need, but it's just too complicated.

* GemDrive is much simpler. It's intended for self-hosters to easily run on
  their own devices.
* WebDAV uses XML. GemDrive uses simple JSON.
* WebDAV requires special HTTP methods.
* WebDAV implementations don't seem to be very compatible with each other.
* GemDrive doesn't specify file locking. Something along these lines may be
  added in the future, depending on real-world needs. So far I haven't needed
  it.


# Amazon S3

* S3 requires a complicated auth implementation which doesn't lend itself to
  web apps.
* S3 is object storage, and as such can't support partial file writes.
* Protocol itself is clunky.
* Not open source. though there are open source implementations such as minio.


# Google Drive

In practice, Google Drive is probably the most successful example of what
GemDrive aims to accomplish, which is providing user-controlled filesystem
storage for web apps. Many popular apps have added support for Google Drive.

Unfortunately, it has some issues:

* [No longer][7] supports hosting websites.
* Has [weird limitations][5] that are annoying/impossible to develop around.
  Note that I believe these mostly exist for security reasons, but I think it's
  possible to have both security and functionality for cloud storage.
* Not open source/can't self-host.


# Dropbox

Dropbox actually provides some really nice libraries for using as a backend for
app storage. I'm surprised more developers haven't built their apps against it.
There may be limitations I'm not aware of.

These are the ones I do know:

* [No longer][6] supports hosting websites.
* Not open source/can't self-host.


# SFTP

* SFTP doesn't work in browser apps.

# NFS

* NFS doesn't work in browser apps.


# FTP

* FTP doesn't work in browser apps.


[0]: https://github.com/remotestorage

[1]: https://github.com/solid/

[2]: https://github.com/gemdrive/gemdrive-ro-server-js

[3]: https://github.com/gemdrive/gemdrive-go



[5]: https://gdrivemusic.com/help

[6]: https://help.dropbox.com/files-folders/share/public-folder

[7]: https://workspaceupdates.googleblog.com/2015/08/deprecating-web-hosting-support-in.html
