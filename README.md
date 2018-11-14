# lerna-travis-typescript-yarn-monorepo

[![Travis CI](https://travis-ci.org/haraldF/lerna-travis-typescript-yarn-monorepo.svg?branch=master)](https://travis-ci.org/haraldF/lerna-travis-typescript-yarn-monorepo)

Demo repository how to have a TypeScript monorepo using yarn workspaces published with lerna from Travis CI

## Rationale

When developing multiple [TypeScript](http://www.typescriptlang.org/) based modules together in a monorepo, it's often
non-obvious how to tie it all together.

## Solution

The best solution I found so far is:

* Use [yarn workspaces](https://yarnpkg.com/lang/en/docs/workspaces/). This automagically links your npm packages together and allows their parallel development without having to resolve to hacks like `npm link`.
* Use [lerna](https://lernajs.io/) for publishing. `lerna` automagically figures out which packages have changed since the last publish, does the tagging and bumping of versions and uploading for us. No custom script magic required.
* Use [Travis CI](https://travis-ci.org/) for continuous integration and publishing. Its defaults are sane and it integrates beautifully with GitHub.

This workflow is supported by most tools and packages.

## Developing

Clone this repo, then run `yarn install` to install the dependencies.

**Note**: For [Visual Studio Code](https://code.visualstudio.com/) to work correctly, `yarn install` must be run once. After that, code completion should run out of the box.

Try running `yarn test`. Then modify the `lttym-lib` library and run `yarn test` again - voila, your modifications to the library are automagically picked up by the app without any hacks required. Happy parallel developing :)

## Alternatives

* `lerna bootstrap` would be an alternative to `yarn workspaces`, but I found it more natural to let my package manager handle all package dependencies, including the dependencies between the packages within the monorepo. Also, I perceive `yarn`'s performance a bit faster than `lerna`'s.
* Any other CI system besides Travis CI. Pretty much any CI system that supports injecting secure tokens to a build should do fine.
