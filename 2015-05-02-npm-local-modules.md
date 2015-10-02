# Build modular application with npm local modules

## Rationale

Most large javascript application built using npm modules have the same problem: tons of javascript files stored in deeply nested folders.

You will [definitely](https://gist.github.com/branneman/8048520) end up with ugly require such as

```js
var User = require('../../../../models/User');
```

This sucks. Relative requires do not scale well. What you want instead is:

```js
var User = require('app-models').User;
```

> Hey but that means I need to publish my module, use private modules or host it on a separate git repo?!

Nope.

## Solution

npm supports a variety of ways to declare module dependencies, the least known being the local one.

Using local modules, here is what your package.json can look like:

```json
{
 "name": "my-app",
 "dependencies": {
   "lodash": "^2.0.0",
   "my-module": "file:local_modules/my-module",
 }
}
```

That's about it. Simply declare the path to your local module, prefixed with _"file:"_.

You can even npm install them and have the package.json written for you:

```sh
npm install --save ./path_to_my/custom_module
```

This will copy your local module into the node_modules folder, the same way it would do with a repository or git hosted module.

Obviously, this means your local module should have its own package.json, README.md, etc, like any regular node module.

This also means that you need to update your node_module copy every time you modify your original code. This can be achieved by either:

```sh
rm -rf node_module/custom_module && npm install
```

which is easy when you are working on the custom module, but the cleaner way to update it is by bumping its package.json version and running `npm update`

## Additional benefits

You have to force yourself writing coherent, self contained, decoupled modules. That's a good thing.

You realized some of your submodule could be a 3rd party library, you think about publishing it. Cool! That is now super easy as you already have a module structure, including its own package.json, and your app uses it the same way it would if the module was published. Just move it to its own repo, publish, change your app dependency, profit. No code refactor involved.

## Example

Just play with this simple repo: [https://github.com/ArnaudRinquin/local_modules_poc](https://github.com/ArnaudRinquin/local_modules_poc)

## Caveats

Local module proper handling only comes with [npm 2.9.0](https://github.com/iojs/io.js/commit/7c89c4c7acdaa2035ec42195ade689419209c2fd#diff-4ac32a78649ca5bdd8e0ba38b7006a1eR21), which comes with [io.js 2.0.0](https://github.com/iojs/io.js/blob/71dc7152eee93c70ab1a078b07c620e6d67c97fe/CHANGELOG.md#notable-changes). You can still use local modules with previous versions of npm, but the update mechanism is [broken](https://github.com/npm/npm/issues/7426).

_Update:_ These caveats are not relevant anymore as you are probably using the `iojs + original node` [converged version](https://nodejs.org/en/blog/release/v4.0.0/): `node 4.0.0+`. I you don't, you should.
