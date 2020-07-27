# node.bcrypt.js
[![Build Status](https://travis-ci.org/kelektiv/node.bcrypt.js.svg?branch=master)](https://travis-ci.org/kelektiv/node.bcrypt.js)
[![Dependency Status](https://david-dm.org/kelektiv/node.bcrypt.js.svg)](https://david-dm.org/kelektiv/node.bcrypt.js)

A library to help you hash passwords.

You can read about [bcrypt in Wikipedia][bcryptwiki] as well as in the following article:
[How To Safely Store A Password][codahale]

## If You Are Submitting Bugs or Issues

Verify that the node version you are using is a _stable_ version; it has an even major release number. Unstable versions are currently not supported and issues created while using an unstable version will be closed.

If you are on a stable version of node, please provide a sufficient code snippet or log files for installation issues. The code snippet does not require you to include confidential information. However, it must provide enough information such that the problem can be replicable. Issues which are closed without resolution often lack required information for replication.


## Version Compatibility

| Node Version   |   Bcrypt Version  |
| -------------- | ------------------|
| 0.4            | <= 0.4            |
| 0.6, 0.8, 0.10 | >= 0.5            |
| 0.11           | >= 0.8            |
| 4              | <= 2.1.0          |
| 8              | >= 1.0.3 < 4.0.0 |
| 10, 11         | >= 3              |
| 12             | >= 3.0.6          |

`node-gyp` only works with stable/released versions of node. Since the `bcrypt` module uses `node-gyp` to build and install, you'll need a stable version of node to use bcrypt. If you do not, you'll likely see an error that starts with:

```
gyp ERR! stack Error: "pre" versions of node cannot be installed, use the --nodedir flag instead
```

## Security Issues And Concerns

> Per bcrypt implementation, only the first 72 bytes of a string are used. Any extra bytes are ignored when matching passwords. Note that this is not the first 72 *characters*. It is possible for a string to contain less than 72 characters, while taking up more than 72 bytes (e.g. a UTF-8 encoded string containing emojis).

As should be the case with any security tool, this library should be scrutinized by anyone using it. If you find or suspect an issue with the code, please bring it to my attention and I'll spend some time trying to make sure that this tool is as secure as possible.

To make it easier for people using this tool to analyze what has been surveyed, here is a list of BCrypt related security issues/concerns as they've come up.

* An [issue with passwords][jtr] was found with a version of the Blowfish algorithm developed for John the Ripper. This is not present in the OpenBSD version and is thus not a problem for this module. HT [zooko][zooko].
* Versions `< 5.0.0` suffer from bcrypt wrap-around bug and _will truncate passwords >= 255 characters leading to severely weakened passwords_. Please upgrade at  earliest. See [this wiki page][wrap-around-bug] for more details.
* Versions `< 5.0.0` _do not handle NUL characters inside passwords properly leading to all subsequent characters being dropped and thus resulting in severely  weakened passwords_. Please upgrade at earliest. See [this wiki page][improper-nuls] for more details.

## Compatibility Note

This library supports `$2a$` and `$2b$` prefix bcrypt hashes. `$2x$` and `$2y$` hashes are specific to bcrypt implementation developed for John the Ripper. In theory, they should be compatible with `$2b$` prefix.

Compatibility with hashes generated by other languages is not 100% guaranteed due to difference in character encodings. However, it should not be an issue for most cases.

### Migrating from v1.0.x

Hashes generated in earlier version of `bcrypt` remain 100% supported in `v2.x.x` and later versions. In most cases, the migration should be a bump in the `package.json`.

Hashes generated in `v2.x.x` using the defaults parameters will not work in earlier versions.

## Dependencies

* NodeJS
* `node-gyp`
 * Please check the dependencies for this tool at: https://github.com/nodejs/node-gyp
  * Windows users will need the options for c# and c++ installed with their visual studio instance.
  * Python 2.x
* `OpenSSL` - This is only required to build the `bcrypt` project if you are using versions <= 0.7.7. Otherwise, we're using the builtin node crypto bindings for seed data (which use the same OpenSSL code paths we were, but don't have the external dependency).

## Install via NPM

```
npm install bcrypt
```
***Note:*** OS X users using Xcode 4.3.1 or above may need to run the following command in their terminal prior to installing if errors occur regarding xcodebuild: ```sudo xcode-select -switch /Applications/Xcode.app/Contents/Developer```

_Pre-built binaries for various NodeJS versions are made available on a best-effort basis._

Only the current stable and supported LTS releases are actively tested against. Please note that there may be an interval between the release of the module and the availabilty of the compiled modules.

Currently, we have pre-built binaries that support the following platforms:

1. Windows x32 and x64
2. Linux x64 (GlibC targets only). Pre-built binaries for MUSL targets such as Apline Linux are not available.
3. macOS

If you face an error like this:

```
node-pre-gyp ERR! Tried to download(404): https://github.com/kelektiv/node.bcrypt.js/releases/download/v1.0.2/bcrypt_lib-v1.0.2-node-v48-linux-x64.tar.gz
```

make sure you have the appropriate dependencies installed and configured for your platform. You can find installation instructions for the dependencies for some common platforms [in this page][depsinstall].

## Usage

### async (recommended)

```javascript
const bcrypt = require('bcrypt');
const saltRounds = 10;
const myPlaintextPassword = 's0/\/\P4$$w0rD';
const someOtherPlaintextPassword = 'not_bacon';
```

#### To hash a password:

Technique 1 (generate a salt and hash on separate function calls):

```javascript
bcrypt.genSalt(saltRounds, function(err, salt) {
    bcrypt.hash(myPlaintextPassword, salt, function(err, hash) {
        // Store hash in your password DB.
    });
});
```

Technique 2 (auto-gen a salt and hash):

```javascript
bcrypt.hash(myPlaintextPassword, saltRounds, function(err, hash) {
    // Store hash in your password DB.
});
```

Note that both techniques achieve the same end-result.

#### To check a password:

```javascript
// Load hash from your password DB.
bcrypt.compare(myPlaintextPassword, hash, function(err, result) {
    // result == true
});
bcrypt.compare(someOtherPlaintextPassword, hash, function(err, result) {
    // result == false
});
```

[A Note on Timing Attacks](#a-note-on-timing-attacks)

### with promises

bcrypt uses whatever Promise implementation is available in `global.Promise`. NodeJS >= 0.12 has a native Promise implementation built in. However, this should work in any Promises/A+ compliant implementation.

Async methods that accept a callback, return a `Promise` when callback is not specified if Promise support is available.

```javascript
bcrypt.hash(myPlaintextPassword, saltRounds).then(function(hash) {
    // Store hash in your password DB.
});
```
```javascript
// Load hash from your password DB.
bcrypt.compare(myPlaintextPassword, hash).then(function(result) {
    // result == true
});
bcrypt.compare(someOtherPlaintextPassword, hash).then(function(result) {
    // result == false
});
```

This is also compatible with `async/await`

```javascript
async function checkUser(username, password) {
    //... fetch user from a db etc.

    const match = await bcrypt.compare(password, user.passwordHash);

    if(match) {
        //login
    }

    //...
}
```

### sync

```javascript
const bcrypt = require('bcrypt');
const saltRounds = 10;
const myPlaintextPassword = 's0/\/\P4$$w0rD';
const someOtherPlaintextPassword = 'not_bacon';
```

#### To hash a password:

Technique 1 (generate a salt and hash on separate function calls):

```javascript
const salt = bcrypt.genSaltSync(saltRounds);
const hash = bcrypt.hashSync(myPlaintextPassword, salt);
// Store hash in your password DB.
```

Technique 2 (auto-gen a salt and hash):

```javascript
const hash = bcrypt.hashSync(myPlaintextPassword, saltRounds);
// Store hash in your password DB.
```

As with async, both techniques achieve the same end-result.

#### To check a password:

```javascript
// Load hash from your password DB.
bcrypt.compareSync(myPlaintextPassword, hash); // true
bcrypt.compareSync(someOtherPlaintextPassword, hash); // false
```

[A Note on Timing Attacks](#a-note-on-timing-attacks)

### Why is async mode recommended over sync mode?
If you are using bcrypt on a simple script, using the sync mode is perfectly fine. However, if you are using bcrypt on a server, the async mode is recommended. This is because the hashing done by bcrypt is CPU intensive, so the sync version will block the event loop and prevent your application from servicing any other inbound requests or events. The async version uses a thread pool which does not block the main event loop.

## API

`BCrypt.`

  * `genSaltSync(rounds, minor)`
    * `rounds` - [OPTIONAL] - the cost of processing the data. (default - 10)
    * `minor` - [OPTIONAL] - minor version of bcrypt to use. (default - b)
  * `genSalt(rounds, minor, cb)`
    * `rounds` - [OPTIONAL] - the cost of processing the data. (default - 10)
    * `minor` - [OPTIONAL] - minor version of bcrypt to use. (default - b)
    * `cb` - [OPTIONAL] - a callback to be fired once the salt has been generated. uses eio making it asynchronous. If `cb` is not specified, a `Promise` is returned if Promise support is available.
      * `err` - First parameter to the callback detailing any errors.
      * `salt` - Second parameter to the callback providing the generated salt.
  * `hashSync(data, salt)`
    * `data` - [REQUIRED] - the data to be encrypted.
    * `salt` - [REQUIRED] - the salt to be used to hash the password. if specified as a number then a salt will be generated with the specified number of rounds and used (see example under **Usage**).
  * `hash(data, salt, cb)`
    * `data` - [REQUIRED] - the data to be encrypted.
    * `salt` - [REQUIRED] - the salt to be used to hash the password. if specified as a number then a salt will be generated with the specified number of rounds and used (see example under **Usage**).
    * `cb` - [OPTIONAL] - a callback to be fired once the data has been encrypted. uses eio making it asynchronous. If `cb` is not specified, a `Promise` is returned if Promise support is available.
      * `err` - First parameter to the callback detailing any errors.
      * `encrypted` - Second parameter to the callback providing the encrypted form.
  * `compareSync(data, encrypted)`
    * `data` - [REQUIRED] - data to compare.
    * `encrypted` - [REQUIRED] - data to be compared to.
  * `compare(data, encrypted, cb)`
    * `data` - [REQUIRED] - data to compare.
    * `encrypted` - [REQUIRED] - data to be compared to.
    * `cb` - [OPTIONAL] - a callback to be fired once the data has been compared. uses eio making it asynchronous. If `cb` is not specified, a `Promise` is returned if Promise support is available.
      * `err` - First parameter to the callback detailing any errors.
      * `same` - Second parameter to the callback providing whether the data and encrypted forms match [true | false].
  * `getRounds(encrypted)` - return the number of rounds used to encrypt a given hash
    * `encrypted` - [REQUIRED] - hash from which the number of rounds used should be extracted.

## A Note on Rounds

A note about the cost. When you are hashing your data the module will go through a series of rounds to give you a secure hash. The value you submit there is not just the number of rounds that the module will go through to hash your data. The module will use the value you enter and go through `2^rounds` iterations of processing.

From @garthk, on a 2GHz core you can roughly expect:

    rounds=8 : ~40 hashes/sec
    rounds=9 : ~20 hashes/sec
    rounds=10: ~10 hashes/sec
    rounds=11: ~5  hashes/sec
    rounds=12: 2-3 hashes/sec
    rounds=13: ~1 sec/hash
    rounds=14: ~1.5 sec/hash
    rounds=15: ~3 sec/hash
    rounds=25: ~1 hour/hash
    rounds=31: 2-3 days/hash


## A Note on Timing Attacks

Because it's come up multiple times in this project, and other bcrypt projects, it needs to be said. The bcrypt comparison function is not susceptible to timing attacks. From codahale/bcrypt-ruby#42:

> One of the desired properties of a cryptographic hash function is preimage attack resistance, which means there is no shortcut for generating a message which, when hashed, produces a specific digest.

A great thread on this, in much more detail can be found @ codahale/bcrypt-ruby#43

If you're unfamiliar with timing attacks and want to learn more you can find a great writeup @ [A Lesson In Timing Attacks][timingatk]

However, timing attacks are real. And, the comparison function is _not_ time safe. What that means is that it may exit the function early in the comparison process. This happens because of the above. We don't need to be careful that an attacker is going to learn anything, and our comparison function serves to provide a comparison of hashes, it is a utility to the overall purpose of the library. If you end up using it for something else we cannot guarantee the security of the comparator. Keep that in mind as you use the library.

## Hash Info

The characters that comprise the resultant hash are `./ABCDEFGHIJKLMNOPQRSTUVWXYZabcdefghijklmnopqrstuvwxyz0123456789$`.

Resultant hashes will be 60 characters long.

## Testing

If you create a pull request, tests better pass :)

```
npm install
npm test
```

## Credits

The code for this comes from a few sources:

* blowfish.cc - OpenBSD
* bcrypt.cc - OpenBSD
* bcrypt::gen_salt - [gen_salt inclusion to bcrypt][bcryptgs]
* bcrypt_node.cc - me

## Contributors

* [Antonio Salazar Cardozo][shadowfiend] - Early MacOS X support (when we used libbsd)
* [Ben Glow][pixelglow] - Fixes for thread safety with async calls
* [Van Nguyen][thegoleffect] - Found a timing attack in the comparator
* [NewITFarmer][newitfarmer] - Initial Cygwin support
* [David Trejo][dtrejo] - packaging fixes
* [Alfred Westerveld][alfredwesterveld] - packaging fixes
* [Vincent Côté-Roy][vincentr] - Testing around concurrency issues
* [Lloyd Hilaiel][lloyd] - Documentation fixes
* [Roman Shtylman][shtylman] - Code refactoring, general rot reduction, compile options, better memory management with delete and new, and an upgrade to libuv over eio/ev.
* [Vadim Graboys][vadimg] - Code changes to support 0.5.5+
* [Ben Noordhuis][bnoordhuis] - Fixed a thread safety issue in nodejs that was perfectly mappable to this module.
* [Nate Rajlich][tootallnate] - Bindings and build process.
* [Sean McArthur][seanmonstar] - Windows Support
* [Fanie Oosthuysen][weareu] - Windows Support
* [Amitosh Swain Mahapatra][agathver] - $2b$ hash support, ES6 Promise support
* [Nicola Del Gobbo][NickNaso] - Initial implementation with N-API

## License
Unless stated elsewhere, file headers or otherwise, the license as stated in the LICENSE file.

[bcryptwiki]: https://en.wikipedia.org/wiki/Bcrypt
[bcryptgs]: http://mail-index.netbsd.org/tech-crypto/2002/05/24/msg000204.html
[codahale]: http://codahale.com/how-to-safely-store-a-password/
[gh13]: https://github.com/ncb000gt/node.bcrypt.js/issues/13
[jtr]: http://www.openwall.com/lists/oss-security/2011/06/20/2
[depsinstall]: https://github.com/kelektiv/node.bcrypt.js/wiki/Installation-Instructions
[timingatk]: https://codahale.com/a-lesson-in-timing-attacks/
[wrap-around-bug]: https://github.com/kelektiv/node.bcrypt.js/wiki/Security-Issues-and-Concerns#bcrypt-wrap-around-bug-medium-severity
[improper-nuls]: https://github.com/kelektiv/node.bcrypt.js/wiki/Security-Issues-and-Concerns#improper-nul-handling-medium-severity

[shadowfiend]:https://github.com/Shadowfiend
[thegoleffect]:https://github.com/thegoleffect
[pixelglow]:https://github.com/pixelglow
[dtrejo]:https://github.com/dtrejo
[alfredwesterveld]:https://github.com/alfredwesterveld
[newitfarmer]:https://github.com/newitfarmer
[zooko]:https://twitter.com/zooko
[vincentr]:https://twitter.com/vincentcr
[lloyd]:https://github.com/lloyd
[shtylman]:https://github.com/shtylman
[vadimg]:https://github.com/vadimg
[bnoordhuis]:https://github.com/bnoordhuis
[tootallnate]:https://github.com/tootallnate
[seanmonstar]:https://github.com/seanmonstar
[weareu]:https://github.com/weareu
[agathver]:https://github.com/Agathver
[NickNaso]: https://github.com/NickNaso
