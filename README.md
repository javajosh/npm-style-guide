# npm Style Guide

Opinionated ​*npm Style Guide*​ for teams by [De Voorhoede](https://twitter.com/devoorhoede).

[![npm style guide](https://img.shields.io/badge/style%20guide-npm-5ed9c7.svg)](https://github.com/voorhoede/npm-style-guide)

## Purpose

This guide provides a set of rules to better manage, test and build your [npm](https://npmjs.org) modules and project scripts. It should make them

* easier for a new developer to pick up
* reduce friction with different environment configurations
* have a predictable api
* easier to add new scripts

## Table of Contents

* [Use nvm to manage node versions](#use-nvm-to-manage-node-versions)
* [Configure your npm personal info](#configure-your-npm-personal-info)
* [Use `save exact` option](#use-save-exact-option)
* [Specify engines on `package.json`](#specify-engines-on-packagejson)
* [Avoid installing modules globally](#avoid-installing-modules-globally)
* [Use standard script names](#use-standard-script-names)
* [Write atomic scripts](#write-atomic-scripts)
* [Use npm modules for system tasks](#use-npm-modules-for-system-tasks)
* [Avoid shorthand command flags](#avoid-shorthand-command-flags)
* [Group related scripts by prefix](#group-related-scripts-by-prefix)
* [Document your script API](#document-your-script-api)


## Use nvm to manage node versions

### Why?

With [nvm](https://github.com/creationix/nvm) you can have multiple different versions available and switch to the one that suits better your project.

### How?

To install or update nvm, you can use the install script using cURL:

```bash
curl -o- https://raw.githubusercontent.com/creationix/nvm/v0.31.0/install.sh | bash
```

For Windows check [nvm for windows](https://github.com/coreybutler/nvm-windows).


If everything goes well, you can now install a specific node version.

```bash
nvm install stable
nvm install vX.Y.Z
nvm alias default stable
```

It’s also easy when updating a newer version, copying your existing global modules.

```bash
nvm copy-packages <previous-version>
```

[↑ back to Table of Contents](#table-of-contents)


## Configure your npm personal info

### Why?

When creating a new package `npm init` your defaults will be already included on the scaffolding.

### How?

```bash
npm config set init-author-name "{name}"
npm config set init-author-email "{email}"
```

Check [npm config](https://docs.npmjs.com/misc/config#init-module) docs, for more info.

You can use `cat ~/.npmrc` to check your current definitions.  

[↑ back to Table of Contents](#table-of-contents)


## Use `save exact` option

### Why?

By default, installing a package with the `--save` or `--save-dev` option, npm saves the package version with `^` prefix, meaning that will update minor versions if available. While this is a good idea as is, this makes it possible for different developers having different versions of the same package and making it harder to debug if there is inconsistency. Defining the `save-exact` option prevents this. More info [npm config](https://docs.npmjs.com/misc/config#save-exact) docs.

### How?

```bash
npm config set save-exact
```

[↑ back to Table of Contents](#table-of-contents)


## Specify engines on `package.json`

### Why?

Specifying engine versions for your module, warns the user if he is not using a supported version. This is specially important for ensuring npm@3 flat tree dependency on Windows, or ES2015 features that your scripts require on node.

### How?

In `package.json`:
```javascript
"engines" : {
  "node" : "5.10.0",
  "npm" : "3.8.5"
}
```

Preventing the user from using your module is also possible with [check-pkg-engines](https://www.npmjs.com/package/check-pkg-engines).

[↑ back to Table of Contents](#table-of-contents)


## Avoid installing modules globally

NPM first tries globally installed modules before looking for local ones. Globally installed modules are shared between projects and might not match the required version for the project.

### Why?

* Locally installed modules are custom and specific for the project.
* Locally installed modules are directly accessible via [npm scripts](https://docs.npmjs.com/misc/scripts).

### How?

```bash
# recommended: install locally
npm install --save-dev grunt-cli grunt
```
and use in `package.json`:
```json
"scripts": {
  "icons": "grunt grunticon"
}
```
```bash
# avoid: don't install modules globally
npm install -g grunt-cli grunt
```

[↑ back to Table of Contents](#table-of-contents)


## Use npm modules for system tasks

(@javajosh: Okay, I don't really agree with this one. I think it's perfectly okay to think in terms of "strings that you emit into the surrounding operating system", in which case it makes more sense to think about all the platforms you want to support, and provide the equivalent string for all of them. This also keeps things simpler, and is much easier to debug. When the inevitable variations come in, thanks to other elements of users environments being different, it is much easier to confront that family of problems as a tree of related strings, as opposed to...whatever it is when you do it this way. (Partly this is also to preserve my own bias that "module" should be bigger than just "mkdir"! There is more than a little similarity to the ORM vs "SQL String" spectrum of solutions to database interaction. Personally, I really appreciate Strings. It is okay if they are somewhat coupled to the program that consumes them! That is inevitable. In fact, I would even argue that it's a good thing because you can characterize a module as triggered when a string gets too long...)


```
macos (bsd) (tcp/ip, dns, net, clock, disk, usb, bt, cam, mic, key, touch, mouse, screen, proc)
	safari chrome
	terminal iterm (oh-my-zsh, autojump, ag, nvalt)
	homebrew
	erlang
	java/kotlin/clojure/groovy
	maven/gradle
	nvm/node/npm/yarn
	virtualbox + alpine,nixos,freebsd,win10
	postgresql
	bootsrap, react/redux/angular/vue, Elixer
	Dropwizard (Jetty), Spring, 
	c/rust/go/clang/llvm/gcc/make
	python3
	redis
	sshd
	nginx
	sublime
	intellij
	vim
```

Each of these, in turn, has many versions and are highly configurable. Wouldn't it be nice if we could just have all the versions and easily change configuration when we need to? The remarkable fact is that we can, but it requires the adoption of some habits and tools. Users cannot trust applications to do the right thing! In fact, quite the opposite. And its time for those that profit from this quiet exfiltration of data to explain themselves (says the naive idealist). But, the more seasoned practical person sees an opportunity: there is an asymmetry here, and a substantial one. That means that there is a big product.

What if applications were routinely written in such a way that allowed multiple versions to be stored? Maven already does this, by just caching, locally, all versions of all libraries you or your dependencies ever required, using the file-system as a simple but effective database. It's also rather wasteful (how wasteful depends on the product, but this is a good reason to keep your bundles as small as possible). With open source libraries, especially using git, you can do better - try to build everything from scratch. If you can make it work, now you can move the working directory around at will. If the project supplies a Dockerfile for a working development environment, by all means use it! by a) depending on your locally checked out and built version, and b) getting the library to build. Now you can have any version you want (that you have in the git repo) by just changing the contents of the working directory with git commands.

### Why?

When you use system specific commands like `rm -rf` or `&&`, you are locking your tasks to your current operating system. If you want to make your scripts work everywhere think about Windows developers also.

### How?

Use npm modules with node that mimic the same tasks but are system agnostic. Some examples:

* create directory (`mkdir` / `mkdir -p`) -> [`mkdirp`](https://www.npmjs.com/package/mkdirp)
* remove files and directories (`rm ...`) -> [`rimraf`](https://www.npmjs.com/package/rimraf)
* copy files (`cp ...`) -> [`ncp`](https://www.npmjs.com/package/ncp)
* run multiple scripts in sequence (`... && ...`) or in parallel (`... & ...`) -> [`npm-run-all`](https://www.npmjs.com/package/npm-run-all)
* set environment variable (`ENV_VAR = ...`) -> [`cross-env`](https://www.npmjs.com/package/cross-env)

[↑ back to Table of Contents](#table-of-contents)


## Avoid shorthand command flags

npm and npm modules with a command-line interface support different options using fully written out and / or shorthand flags. For instance, instead of `npm install --save-dev` you can use the shorter `npm i -D`. For `npm test` you can use simply `npm t`. But `npm start` is not the same as `npm s`, as that's an alias for `npm search`. So while you can use these shorthands in your daily routine, you should avoid them in scripts and documentation shared with other developers.  

### Why?

* Shorthand flags can only be understood by developers who know the modules and options well.
* Fully written out command options help in writing self documented scripts.
* Fully written out command options make scripts more accessible to other developers.

### How?

Always prefer fully written command flags over shorthand. Example using [uglifyjs](https://www.npmjs.com/package/uglify-js):

```bash
# recommended
uglify index.js --compress --mangle --reserved '$' --output index.min.js
```

```bash
# avoid
uglifyjs index.js -c -m -r '$' -o index.min.js
```

[↑ back to Table of Contents](#table-of-contents)

## Use standard script names

npm lets you define custom scripts. You can give these scripts any name you like, but you should stick to standard names when you can.

### Why?

Using standard script names creates a predictable script API, which makes your project easier to use by other developers. When used consistently between projects a user doesn't even need to read the documentation or `package.json` to know which scripts are available.

### How?

`npm start` and `npm test` are predefined aliases for custom scripts. In addition use of `build`, `deploy` and `watch` are widely spread within the developer community. You should use these script names as follows:

* **`npm run build`** to create a distribution of your project (mostly for production).
* **`npm run deploy`** to put your project on a host environment.
* **`npm start`** (alias for `npm run start`) to start a web server (defaults to `node server.js`).
* **`npm test`** (alias for `npm run test`) to run project's entire test suite.
* **`npm run watch`** to run other scripts on files changes.

In `package.json`:
```javascript
/* recommended: standard script names */
{
  "scripts": {
    "build": "...",
    "deploy": "...",
    "start": "...",
    "test": "...",
    "watch": "..."
  }
}

/* avoid: */
{
  "scripts": {
    "bundle": "...",
    "upload": "...",
    "serve": "...",
    "check": "...",
    "watcher": "..."
  }
}
```

[↑ back to Table of Contents](#table-of-contents)


## Write atomic scripts

Each script should be only responsible for one action.

### Why?

* Atomic scripts are easy to read and understand.
* Atomic scripts are easy to reuse.

### How?

Separate each step of the script to an individual script. For example a "generate icon" script can be split into atomic script like "clean directory", "optimize SVGs", "generate PNGs" and "generate data-uris for SVGs".

[↑ back to Table of Contents](#table-of-contents)

## Group related scripts by prefix

Bundle your scripts with a prefix so you can execute them all at once.

### Why?

* Bundling helps keeping your scripts organized.
* Tasks grouped by prefix can be easily executed with one command.
* Your high-level script API remains unchanged when scripts are added, removed or renamed.

### How?

In `package.json`:
```javascript
/* recommended: group related scripts by prefix */
scripts: {
	"test": "npm run test:eslint && npm run test:unit && npm run test:e2e",
	"test:eslint": "eslint src/**/*.js",
	"test:unit": "tape --require dist/index.js src/**/*.test.js",
	"test:e2e": "karma start test/config.js"
}

/* avoid */
scripts: {
	"eslint": "eslint src/**/*.js",
	"tape": "tape --require dist/index.js src/**/*.test.js",
	"karma": "karma start test/config.js",
}
```

Bundled scripts can be executed (in parallel or in sequence) using [npm-run-all](https://www.npmjs.com/package/npm-run-all):

```javascript
/* recommended: use `npm-run-all` to run all bundled scripts */
scripts: {
	"test": "npm-run-all test:*",
	"test:eslint": "eslint src/**/*.js",
	"test:unit": "tape --require dist/index.js src/**/*.test.js",
	"test:e2e": "karma start test/config.js"
}
```

[↑ back to Table of Contents](#table-of-contents)


## Document your script API

### Why?

* Documentation provides developers with a high level overview to the script, without the need to go through all its code. This makes a module more accessible and easier to use.
* Documentation formalises the API.

### How?

Document your script API in the project's README.md or CONTRIBUTING.md as those are the first places contributors will look.
Describe what each task does using a simple table:

```markdown
`npm run ...` | Description
---|---
task | What it does as a plain human readable description.
```

An example:

`npm run ...` | Description
---|---
`build` | Compile, bundle and minify all CSS and JS files..
`build:css` | Compile, autoprefix and minify all CSS files to `dist/index.css`.
`build:js` | Compile, bundle and minify all JS files to `dist/index.js`.
`start` | Starts a server on `http://localhost:3000`.
`test` | Run all unit and end-to-end tests.

[↑ back to Table of Contents](#table-of-contents)

---

## License

[![CC0](http://mirrors.creativecommons.org/presskit/buttons/88x31/svg/cc-zero.svg)](https://creativecommons.org/publicdomain/zero/1.0/)

[De Voorhoede](https://twitter.com/devoorhoede) waives all rights to this work worldwide under copyright law, including all related and neighboring rights, to the extent allowed by law.

You can copy, modify, distribute and perform the work, even for commercial purposes, all without asking permission.
