# npm-logical-tree [![npm version](https://img.shields.io/npm/v/npm-logical-tree.svg)](https://npm.im/npm-logical-tree) [![license](https://img.shields.io/npm/l/npm-logical-tree.svg)](https://npm.im/npm-logical-tree) [![Travis](https://img.shields.io/travis/npm/logical-tree.svg)](https://travis-ci.org/npm/logical-tree) [![AppVeyor](https://ci.appveyor.com/api/projects/status/github/npm/logical-tree?svg=true)](https://ci.appveyor.com/project/npm/logical-tree) [![Coverage Status](https://coveralls.io/repos/github/npm/logical-tree/badge.svg?branch=latest)](https://coveralls.io/github/npm/logical-tree?branch=latest)

[`npm-logical-tree`](https://github.com/npm/npm-logical-tree) is a Node.js
library that takes the contents of a `package.json` and `package-lock.json` (or
`npm-shrinkwrap.json`) and returns a nested tree data structure representing the
logical relationships between the different dependencies.

## Install

`$ npm install npm-logical-tree`

## Table of Contents

* [Example](#example)
* [Contributing](#contributing)
* [API](#api)
  * [`logicalTree`](#logical-tree)
  * [`logicalTree.node`](#make-node)
  * [`tree.isRoot`](#is-root)
  * [`tree.addDep`](#add-dep)
  * [`tree.delDep`](#del-dep)
  * [`tree.getDep`](#get-dep)
  * [`tree.path`](#path)
  * [`tree.hasCycle`](#has-cycle)
  * [`tree.forEach`](#for-each)
  * [`tree.forEachAsync`](#for-each-async)

### Example

```javascript
const fs = require('fs')
const logicalTree = require('npm-logical-tree')

const pkg = require('./package.json')
const pkgLock = require('./package-lock.json')

logicalTree(pkg, pkgLock)
// returns:
LogicalTree {
  name: 'npm-logical-tree',
  version: '1.0.0',
  address: null,
  optional: false,
  dev: false,
  bundled: false,
  resolved: undefined,
  integrity: undefined,
  requiredBy: Set { },
  dependencies:
   Map {
     'foo' => LogicalTree {
       name: 'foo',
       version: '1.2.3',
       address: 'foo',
       optional: false,
       dev: true,
       bundled: false,
       resolved: 'https://registry.npmjs.org/foo/-/foo-1.2.3.tgz',
       integrity: 'sha1-rYUK/p261/SXByi0suR/7Rw4chw=',
       dependencies: Map { ... },
       requiredBy: Set { ... },
     },
     ...
  }
}
```

### Contributing

The npm team enthusiastically welcomes contributions and project participation!
There's a bunch of things you can do if you want to contribute! The [Contributor
Guide](CONTRIBUTING.md) has all the information you need for everything from
reporting bugs to contributing entire new features. Please don't hesitate to
jump in if you'd like to, or even ask us questions if something isn't clear.

All participants and maintainers in this project are expected to follow [Code of
Conduct](CODE_OF_CONDUCT.md), and just generally be excellent to each other.

Please refer to the [Changelog](CHANGELOG.md) for project history details, too.

Happy hacking!

### API

#### <a name="logical-tree"></a> `> logicalTree(pkg, lock) -> LogicalTree`

Calculates a logical tree based on a matching `package.json` and
`package-lock.json` pair. A "logical tree" is a fully-nested dependency graph
for an npm package, as opposed to a physical tree which might be flattened.

`logical-tree` will represent deduplicated/flattened nodes using the same object
throughout the tree, so duplication can be checked by object identity.

##### Example

```javascript
const pkg = require('./package.json')
const pkgLock = require('./package-lock.json')

logicalTree(pkg, pkgLock)
// returns:
LogicalTree {
  name: 'npm-logical-tree',
  version: '1.0.0',
  address: null,
  optional: false,
  dev: false,
  bundled: false,
  resolved: undefined,
  integrity: undefined,
  requiredBy: Set { },
  dependencies:
   Map {
     'foo' => LogicalTree {
       name: 'foo',
       version: '1.2.3',
       address: 'foo',
       optional: false,
       dev: true,
       bundled: false,
       resolved: 'https://registry.npmjs.org/foo/-/foo-1.2.3.tgz',
       integrity: 'sha1-rYUK/p261/SXByi0suR/7Rw4chw=',
       requiredBy: Set { ... },
       dependencies: Map { ... }
     },
     ...
  }
}
```

#### <a name="make-node"></a> `> logicalTree.node(name, [address, [opts]]) -> LogicalTree`

Manually creates a new LogicalTree node.

##### Options

* `opts.version` - version of the node.
* `opts.optional` - is this node an optionalDep?
* `opts.dev` - is this node a devDep?
* `opts.bundled` - is this bundled?
* `opts.resolved` - resolved address.
* `opts.integrity` - SRI string.

##### Example
```javascript
logicalTree.node('hello', 'subpath:to:@foo/bar', {dev: true})
```

#### <a name="is-root"></a> `> tree.isRoot`

Boolean value indicating whether the tree (node) is the root node or not.

##### Example
```javascript
tree.isRoot // true
```

#### <a name="add-dep"></a> `> tree.addDep(node)`

Adds a logical dependency (such as one created by [`tree.node`](#make-node)) to the tree's `dependencies` map. Returns the logicalTree instance.

##### Example
```javascript
const node = logicalTree.node('hello', 'hello')
tree.addDep(node)
```

#### <a name="del-dep"></a> `> tree.delDep(node)`

Deletes a logical dependency from the tree's `dependencies` map. Returns the tree instance.

##### Example
```javascript
const node = logicalTree.getDep('hello')
tree.delDep(node)
```

#### <a name="get-dep"></a> `> tree.getDep(name)`

Returns a logical dependency from the tree's `dependencies` map.

##### Example
```javascript
const node = logicalTree.getDep('hello')
```

#### <a name="path"></a> `> tree.path([prefix])`

Returns the physical path of the node.
An optional prefix (eg. in order to receive an absolute path) can be provided.

##### Example
```javascript
const node = logicalTree.getDep('hello')
node.path() 
// node_modules/foo/node_modules/bar/node_modules/hello
node.path('/home/myHomeFolder/moduleName/') 
// /home/myHomeFolder/moduleName/node_modules/foo/node_modules/bar/node_modules/hello
```

#### <a name="has-cycle"></a> `> tree.hasCycle()`

Returns true if there is a dependency cycle including the current node.
Will not check if there is an internal 'unrelated' cycle within the node's children.

##### Example
```javascript
const a = logicalTree.getDep('a')
const a.hasCycle() // true
// a ===depends-on===> b ===depends-on===> c ===depends-on===> a
```

#### <a name="for-each"></a> `> tree.forEach(fn)`

Executes `fn` once for each node in the tree.
`fn` receives the node object and a `cb` to call when it is done.

##### Example
```javascript
let flattened = {}
logicalTree.forEach((node, cb) => {
  const path = node.path()
  flattened[path] = node
  cb()
})
```

#### <a name="for-each-async"></a> `> tree.forEachAsync(fn, opts)`

Returns a `Promise` resolved once `fn` is executed asynchronously for every node in the tree.
`fn` receives the node object and a `cb` to call when it is done.
`fn` is expected to return a `Promise`, and indeed `cb` here returns a `Promise` as well.

##### Options

* `opts.Promise` - custom `Promise` implementation

##### Example
```javascript
let manifests = {}
await logicalTree.forEach(async (node, cb) => {
  const path = node.path()
  manifests[path] = await pacote.manifest(`${node.name}@${node.version}`)
  await cb()
})
console.log('Yay, we are done!')
```
