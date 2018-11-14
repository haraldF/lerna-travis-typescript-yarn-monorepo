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

## Developing

Clone this repo, then run `yarn install` to install the dependencies.

**Note**: For [Visual Studio Code](https://code.visualstudio.com/) to work correctly, `yarn install` must be run once. After that, code completion should run out of the box.

Try running `yarn test`. Then modify the `lttym-lib` library and run `yarn test` again - voila, your modifications to the library are automagically picked up by the app without any hacks required. Happy parallel developing :)

## Publishing

In order to have reproducable and stable environment for publishing, it must happen from within CI. A developer's machine might have local changes, uncommitted files, or other fancy settings that might affect the generated packages.

CI systems generally use tagged docker images and from the log, we can easily reconstruct how a package was built and published. It also reduces the risk of accidentally publishing a package without tagging or bumping the version number correctly.

What must also be given is that the token used to authenticate to the server must be secure. For this, one must define a secure environment variable called `NPM_TOKEN` in the `Travis CI` configuration.

To create a new release, run:

```sh
npx lerna version patch
```

This bumps the version of all the packages that were modified since the last publish and pushes the commits including the git tags to GitHub.

Travis CI is configured to listen to new git tags (see `.travis.yml`) and will do the actual publishing automagically using lerna.

## Magic

### Directory Structure

Let's start with the directory structure - the defacto standard for `lerna` based repositories is to have one "super" `package.json` in the root directory and the individual packages in a `packages` directory.

### Yarn Workspaces

The root `package.json contains the following:

```json
  "workspaces": [
    "packages/*"
  ],
```

This indicates to `yarn` that it's a workspace. Packages within a workspace can depend on each other and yarn will link it all correctly together.

### Packages

The packages in the `packages` directory are pretty standard node.js packages.

In order to ease packaging, the TypeScript compiler `tsc` is executed in the `prepare` phase of the package, so when `lerna` packs it all together, the package is automagically transpiled to JavaScript.

A note about `.gitignore` and `.npmignore`: It's good practice to have TypeScript files in git, but not in the generated package and JavaScript files in the generated package but not in git. That's why `.gitignore` ignores all `*.js` files and `.npmignore` ignores all `*.ts` files.

### Lerna

The `lerna.json` file is also pretty standard, the only exception is the custom registry. Since I don't want to publish dummy packages to npmjs.com, I've opened a dummy repository on [bintray](https://bintray.com).

### Travis (.travis.yml)

Unfortunately, Travis CI comes with an old version of `yarn`, so it's upgraded in `before_install`.

In `before_deploy`, the custom `.npmrc-publish` configuration file is copied to the user's home directory to make it available system wide.

The custom `.npmrc` contains the following magic line:

```
//api.bintray.com/npm/haraldf/npm-testing/:_authToken=${NPM_TOKEN}
```

This is described in the npm documentation: [https://docs.npmjs.com/using-private-packages-in-a-ci-cd-workflow](https://docs.npmjs.com/using-private-packages-in-a-ci-cd-workflow)

Instead of checking our secret token into git, where it would be world readable, the token is consumed via the environment variable `NPM_TOKEN`. In the Travis CI, this environment variable can be documented as secret token and won't be displayed to the user.

## Alternatives

* `lerna bootstrap` would be an alternative to `yarn workspaces`, but I found it more natural to let my package manager handle all package dependencies, including the dependencies between the packages within the monorepo. Also, I perceive `yarn`'s performance a bit faster than `lerna`'s.
* Any other CI system besides Travis CI. Pretty much any CI system that supports injecting secure tokens to a build should do fine.

## There are better ways to do it!

Great! Please feel free to submit PRs if you think that something can be solved more elegantly, or leave a bug report if you think something is not right.
