# abstract-pull-git-repo

git repo interface using
[pull-streams](https://github.com/dominictarr/pull-stream/).

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

#### `repo.hasObject(hash, cb(err, bool))`

Get whether the repo contains the given git object

- `hash`: git sha1 hash of the object to check the presence of
- `err`: error doing the lookup, if any
- `bool`: truthy if the repo contains the given object

#### `repo.getObject(hash, cb(err, object))`

Get a git object from the repo

- `hash`: sha1 hash of the git object to get
- `object`: the git object, or falsy if it is not present

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

## Test Suite

Use this repo's test suite to test your own git repo implementation:

```js
var tests = require('abstract-pull-git-repo/tests')
var Repo = require('your-custom-git-repo-implementation')
tests(require('tape'), function () { return new Repo() })
```

## Implementations

- [memory-pull-git-repo](https://github.com/clehner/memory-pull-git-repo)

See also keyword
[`pull-git-repo`](https://www.npmjs.com/browse/keyword/pull-git-repo) on npm

## License

Copyright (c) 2016 Charles Lehner

Usage of the works is permitted provided that this instrument is
retained with the works, so that any entity that uses the works is
notified of this instrument.

DISCLAIMER: THE WORKS ARE WITHOUT WARRANTY.