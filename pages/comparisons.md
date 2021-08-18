# WebDAV

* GemDrive is much simpler. It's intended for self-hosters to easily run on
  their own devices.
* WebDAV uses XML.
* WebDAV requires special HTTP methods.
* WebDAV implementations don't seem to be very compatible with each other.


# Amazon S3

* S3 requires a complicated auth implementation which doesn't lend itself to
  web apps.
* S3 is object storage, and as such can't support partial file writes.


# Inrupt Solid

I'm bullish on Solid, and I think GemDrive compliments it nicely. While
GemDrive aims to provide the minimal necessary filesystem features, Solid
takes a highly integrated top-down approach to provided a storage backend for
web apps.


# remoteStorage

remoteStorage is an excellent protocol that is very similar to GemDrive. The
differences are mostly in the details.

* GemDrive manages to still be simpler.
* GemDrive specifies a way for the server to provide thumbnails and previews.
* GemDrive supports hosting web pages as a first-class use case.


# Google Drive


# Dropbox


# NFS

* NFS doesn't work in browser apps, so it's a non-starter.

# FTP

* FTP doesn't work in browser apps, so it's a non-starter.
