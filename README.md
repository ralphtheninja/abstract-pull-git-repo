# abstract-pull-git-repo

git repo interface using [pull-stream][]s

[pull-stream]: https://github.com/dominictarr/pull-stream/
[memory-pull-git-repo]: https://github.com/clehner/memory-pull-git-repo
[ssb-git-repo]: https://github.com/clehner/ssb-git-repo
[tape]: https://github.com/substack/tape
[keyword]: https://www.npmjs.com/browse/keyword/pull-git-repo
[mango]: https://github.com/axic/git-remote-mango/blob/master/mango.js

## API

git objects are represented by an object with the following properties:

- `type`: the type of the object, one of `["tag", "commit", "tree", "blob"]`
- `length`: the size in bytes of the object
- `read`: `read(abort, next(end, buf))`

  readable stream for an object to add

#### `repo.refs([prefix])`: `read(abort, next(end, {name, hash}))`

Get the repo's refs.

- `prefix`: optional prefix for filtering refs, e.g. `'refs/heads'`
- `read`: readable stream of ref info
- `name`: the name of the ref, e.g. `'refs/heads/master'`
- `hash`: git sha1 hash of the git object that the ref points to

#### `repo.symrefs()`: `read(abort, next(end, {name, ref}))`

Get the repo's symbolic refs. Symbolic refs are like refs that point to
other refs. Usually the only symbolic ref is `HEAD`.

- `read`: readable stream of symbolic ref info
- `name`: the name of the symref, e.g. `'HEAD'`
- `ref`: name of the ref pointed to by the symref, e.g. `'refs/heads/master'`

#### `repo.packs()`: `read(abort, next(end, {packId, idxId}))`

Get the repo's packfiles and their corresponding index files.

- `read`: readable stream of packs info
- `packId`: identifier of the packfile, which can be passed to
    `repo.getPackfile` to retrieve the packfile data
- `idxId`: identifier of the pack index file, which can be passed to
    `repo.getPackIndex` to retrieve the pack index file data

#### `repo.hasObject(hash, cb(err, bool))`

Get whether the repo contains the given git object

- `hash`: git sha1 hash of the object to check the presence of
- `err`: error doing the lookup, if any
- `bool`: truthy if the repo contains the given object

#### `repo.getObject(hash, cb(err, object))`

Get a git object from the repo. If the object is not present, an error should
result.

- `hash`: sha1 hash of the git object to get
- `err`: error getting the object, such as if the object does not exist
- `object`: the retrieved git object

#### `repo.getPackfile(id, cb(err, read))`

Get a packfile from the repo

- `id`: identifier for the packfile as returned by `repo.packs`
- `err`: error if the packfile does not exist or cannot be retrieved
- `read`: readable stream of packfile data

#### `repo.getPackIndex(id, cb(err, read))`

Get an index file for a packfile in the repo

- `id`: identifier for the packfile and its index as returned by `repo.packs`
- `err`: error if the index does not exist or cannot be retrieved
- `read`: readable stream of pack index file data

#### `repo.update(refs, objects, cb(err))`

Update a repo by adding objects to it and/or updating its refs. Can only be
done by the feed owner.

- `refs`: `read(abort, next(end, {name, old, new}))`

  readable stream of ref updates

  - `name`: the name of the ref to update
  - `old`: old value (sha1) of the ref. falsy if the ref is to be created
  - `new`: new value (sha1) of the ref. falsy to delete the ref

- `objects`: `read(abort, next(end, object))`

  readable stream of objects to add to the repo

- `cb`: function called after all objects and refs have been read.

  - `err`: error if updating the refs or reading the objects failed.
    If truthy, the repo will not have been updated

#### `repo.uploadPack(refs, packs, cb(err))`

Update a repo by adding a packfile and index file to it and/or updating its
refs. This may only be done by the feed owner. Implementing this method is
optional since it is only an optimization over pushing individual objects with
`repo.update()`

- `refs`: readable stream of ref updates. Same as in `repo.update(refs, ...)`

- `packs`: `read(abort, next(end, pack))`

  readable stream of packs to add to the repo

  - `pack.pack`: readable stream of packfile data
  - `pack.idx`: readable stream of packfile index data

- `cb`: function called after the packfiles, indexes, and refs have been read.

  - `err`: error if updating the refs or reading the packfiles stream failed.
    If truthy, the repo will not have been updated

## Test Suite

Use this repo's test suite to test your own git repo implementation:

### API

#### `tests.repo(test, repo)`

Test that an empty repo can be pushed to and updated

- `test`: [tape][]-compatible test object
- `repo`: `abstract-pull-git-repo`-compatible repo object

##### Example

```js
var test = require('tape')
var tests = require('abstract-pull-git-repo/tests')
var Repo = require('your-custom-git-repo-implementation')

tests.repo(test, new Repo)
```

#### `tests.repos(test, repoA, getRepoB(cb(err, repoB))`

Test that updates pushed to one repo are visible on the other repo

- `test`: [tape][]-compatible test object
- `repoA`: repo to update
- `repoB`: repo to retrieve updates from
- `getRepoB`: function to get repo to retrieve updates from. May be called more
    than once.
- `err`: error getting a repo, if any

##### Example

```js
var SyncedRepo = require('your-custom-synced-git-repo')
var repo = new SyncedRepo()

tests.repos(test, repo, function (cb) {
  cb(null, new SyncedRepo(repo.id))
})
```

## Implementations

- [memory-pull-git-repo][]
- [ssb-git-repo][]
- [mango][]

See also keyword [`pull-git-repo`][keyword] on npm

## License

Copyright (c) 2016 Charles Lehner

Usage of the works is permitted provided that this instrument is
retained with the works, so that any entity that uses the works is
notified of this instrument.

DISCLAIMER: THE WORKS ARE WITHOUT WARRANTY.
