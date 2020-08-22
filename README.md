# Pathy: a python Path interface for bucket storage

[![Build status](https://travis-ci.com/justindujardin/pathy.svg?branch=master)](https://travis-ci.com/justindujardin/pathy)
[![codecov](https://codecov.io/gh/justindujardin/pathy/branch/master/graph/badge.svg)](https://codecov.io/gh/justindujardin/pathy)
[![Pypi version](https://badgen.net/pypi/v/pathy)](https://pypi.org/project/pathy/)

Pathy is a python package (_with type annotations_) for working with Bucket storage providers. It provides a CLI app for basic file operations between local files and remote buckets. It enables a smooth developer experience by supporting local file-system backed buckets during development and testing. It makes converting bucket blobs into local files a snap with optional local file caching of blobs.

## 🚀 Quickstart

You can install `pathy` from pip:

```bash
pip install pathy
```

The package exports the `Pathy` class and utilities for configuring the bucket storage provider to use. By default Pathy prefers GoogleCloudStorage paths of the form `gs://bucket_name/folder/blob_name.txt`. Internally Pathy can convert GCS paths to local files, allowing for a nice developer experience.

```python
from pathy import Pathy

# Any bucket you have access to will work
greeting = Pathy(f"gs://my_bucket/greeting.txt")
# The blob doesn't exist yet
assert not greeting.exists()
# Create it by writing some text
greeting.write_text("Hello World!")
# Now it exists
assert greeting.exists()
# Delete it
greeting.unlink()
# Now it doesn't
assert not greeting.exists()
```

## 🎛 API

<!-- NOTE: The below code is auto-generated. Update source files to change API documentation. -->
<!-- AUTO_DOCZ_START -->

# Pathy

```python
Pathy(self, args, kwargs)
```

Subclass of `pathlib.Path` that works with bucket APIs.

## exists

```python
Pathy.exists(self: ~PathType) -> bool
```

Returns True if the path points to an existing bucket, blob, or prefix.

## fluid

```python
Pathy.fluid(
    path_candidate: Union[str, Pathy, pathlib.Path],
) -> Union[Pathy, pathlib.Path]
```

Infer either a Pathy or pathlib.Path from an input path or string.

The returned type is a union of the potential `FluidPath` types and will
type-check correctly against the minimum overlapping APIs of all the input
types.

If you need to use specific implementation details of a type, "narrow" the
return of this function to the desired type, e.g.

```python
fluid_path = FluidPath("gs://my_bucket/foo.txt")
# Narrow the type to a specific class
assert isinstance(fluid_path, Pathy), "must be Pathy"
# Use a member specific to that class
print(fluid_path.prefix)
```

## from_bucket

```python
Pathy.from_bucket(bucket_name: str) -> 'Pathy'
```

Initialize a Pathy from a bucket name. This helper adds a trailing slash and
the appropriate prefix.

```python
assert str(Pathy.from_bucket("one")) == "gs://one/"
assert str(Pathy.from_bucket("two")) == "gs://two/"
```

## glob

```python
Pathy.glob(self: ~PathType, pattern) -> Generator[~PathType, NoneType, NoneType]
```

Perform a glob match relative to this Pathy instance, yielding all matched
blobs.

## is_dir

```python
Pathy.is_dir(self: ~PathType) -> bool
```

Determine if the path points to a bucket or a prefix of a given blob
in the bucket.

Returns True if the path points to a bucket or a blob prefix.
Returns False if it points to a blob or the path doesn't exist.

## is_file

```python
Pathy.is_file(self: ~PathType) -> bool
```

Determine if the path points to a blob in the bucket.

Returns True if the path points to a blob.
Returns False if it points to a bucket or blob prefix, or if the path doesn’t
exist.

## iterdir

```python
Pathy.iterdir(self: ~PathType) -> Generator[~PathType, NoneType, NoneType]
```

Iterate over the blobs found in the given bucket or blob prefix path.

## mkdir

```python
Pathy.mkdir(
    self: ~PathType,
    mode: int = 511,
    parents: bool = False,
    exist_ok: bool = False,
) -> None
```

Create a bucket from the given path. Since bucket APIs only have implicit
folder structures (determined by the existence of a blob with an overlapping
prefix) this does nothing other than create buckets.

If parents is False, the bucket will only be created if the path points to
exactly the bucket and nothing else. If parents is true the bucket will be
created even if the path points to a specific blob.

The mode param is ignored.

Raises FileExistsError if exist_ok is false and the bucket already exists.

## open

```python
Pathy.open(
    self: ~PathType,
    mode = 'r',
    buffering = 8192,
    encoding = None,
    errors = None,
    newline = None,
) -> io.IOBase
```

Open the given blob for streaming. This delegates to the `smart_open`
library that handles large file streaming for a number of bucket API
providers.

## owner

```python
Pathy.owner(self: ~PathType) -> Optional[str]
```

Returns the name of the user that owns the bucket or blob
this path points to. Returns None if the owner is unknown or
not supported by the bucket API provider.

## rename

```python
Pathy.rename(self: ~PathType, target: Union[str, ~PathType]) -> None
```

Rename this path to the given target.

If the target exists and is a file, it will be replaced silently if the user
has permission.

If path is a blob prefix, it will replace all the blobs with the same prefix
to match the target prefix.

## replace

```python
Pathy.replace(self: ~PathType, target: Union[str, ~PathType]) -> None
```

Renames this path to the given target.

If target points to an existing path, it will be replaced.

## resolve

```python
Pathy.resolve(self: ~PathType) -> ~PathType
```

Resolve the given path to remove any relative path specifiers.

```python
path = Pathy("gs://my_bucket/folder/../blob")
assert path.resolve() == Pathy("gs://my_bucket/blob")
```

## rglob

```python
Pathy.rglob(self: ~PathType, pattern) -> Generator[~PathType, NoneType, NoneType]
```

Perform a recursive glob match relative to this Pathy instance, yielding
all matched blobs. Imagine adding "\*\*/" before a call to glob.

## rmdir

```python
Pathy.rmdir(self: ~PathType) -> None
```

Removes this bucket or blob prefix. It must be empty.

## samefile

```python
Pathy.samefile(self: ~PathType, other_path: ~PathType) -> bool
```

Determine if this path points to the same location as other_path.

## stat

```python
Pathy.stat(self: ~PathType) -> pathy.client.BucketStat
```

Returns information about this bucket path.

## to_local

```python
Pathy.to_local(
    blob_path: Union[Pathy, str],
    recurse: bool = True,
) -> pathlib.Path
```

Download and cache either a blob or a set of blobs matching a prefix.

The cache is sensitive to the file updated time, and downloads new blobs
as their updated timestamps change.

## touch

```python
Pathy.touch(self: ~PathType, mode: int = 438, exist_ok: bool = True)
```

Create a blob at this path.

If the blob already exists, the function succeeds if exist_ok is true
(and its modification time is updated to the current time), otherwise
FileExistsError is raised.

# BucketStat

```python
BucketStat(self, size: int, last_modified: int) -> None
```

Stat for a bucket item

# use_fs

```python
use_fs(
    root: Optional[str, pathlib.Path, bool] = None,
) -> Optional[pathy.file.BucketClientFS]
```

Use a path in the local file-system to store blobs and buckets.

This is useful for development and testing situations, and for embedded
applications.

# get_fs_client

```python
get_fs_client() -> Optional[pathy.file.BucketClientFS]
```

Get the file-system client (or None)

# use_fs_cache

```python
use_fs_cache(
    root: Optional[str, pathlib.Path, bool] = None,
) -> Optional[pathlib.Path]
```

Use a path in the local file-system to cache blobs and buckets.

This is useful for when you want to avoid fetching large blobs multiple
times, or need to pass a local file path to a third-party library.

# get_fs_cache

```python
get_fs_cache() -> Optional[pathlib.Path]
```

Get the folder that holds file-system cached blobs and timestamps.

# CLI

Pathy command line interface.

**Usage**:

```console
$ [OPTIONS] COMMAND [ARGS]...
```

**Options**:

- `--install-completion`: Install completion for the current shell.
- `--show-completion`: Show completion for the current shell, to copy it or customize the installation.
- `--help`: Show this message and exit.

**Commands**:

- `cp`: Copy a blob or folder of blobs from one...
- `ls`: List the blobs that exist at a given...
- `mv`: Move a blob or folder of blobs from one path...
- `rm`: Remove a blob or folder of blobs from a given...

## `cp`

Copy a blob or folder of blobs from one bucket to another.

**Usage**:

```console
$ cp [OPTIONS] FROM_LOCATION TO_LOCATION
```

**Options**:

- `--help`: Show this message and exit.

## `ls`

List the blobs that exist at a given location.

**Usage**:

```console
$ ls [OPTIONS] LOCATION
```

**Options**:

- `--help`: Show this message and exit.

## `mv`

Move a blob or folder of blobs from one path to another.

**Usage**:

```console
$ mv [OPTIONS] FROM_LOCATION TO_LOCATION
```

**Options**:

- `--help`: Show this message and exit.

## `rm`

Remove a blob or folder of blobs from a given location.

**Usage**:

```console
$ rm [OPTIONS] LOCATION
```

**Options**:

- `-r, --recursive`: Recursively remove files and folders.
- `-v, --verbose`: Print removed files and folders.
- `--help`: Show this message and exit.

<!-- AUTO_DOCZ_END -->
