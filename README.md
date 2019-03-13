# Reproducible PureScript Quickstart

This guide offers a simple set of steps to get up and running with
[PureScript](http://www.purescript.org/) with the goal of building
projects in a way that is reproducible and
consistent regardless of the underlying development environment.

Rather than provide a [ready-made starter app](https://github.com/lumihq/purescript-react-basic),
the objective is to provide some contextual understanding of how to
achieve a streamlined and reliable workflow for your PureScript projects.

The following steps will take you from a blank slate to a functioning
"Hello World" SPA.

You'll want to clone and work inside of this repository when following the
instructions below.

# Step 1: Install Nix

[Nix](https://nixos.org/nix/) is a powerful package manager for Linux
and other Unix systems that makes package management reliable and reproducible.

By using Nix as the starting point for your project, you'll be creating
a sandbox for any underlying node/npm dependencies and avoiding polluting
the global environment of your system.

    $ curl https://nixos.org/nix/install | sh

A Nix channel represents a static set of packages that will not change
in a way that breaks compatibility or affects dependencies. You can find
out what the latest stable channel is
[on the Nix website](https://nixos.wiki/wiki/Nix_Channels). As of today,
the latest stable channel is `18.09`.

    $ nix-channel --add https://nixos.org/channels/nixos-18.09 nixpkgs

A Nix derivation (build action) is a declarative set of build instructions
which define how to build a specific set of dependent packages in a
predictable way. The `default.nix` file included in this repository
defines a derivation to import a recent (stable) version of `nodejs`
and `psc-package`.

Invoking `nix-shell` command will build the dependencies of the derivation
specified, and it will start an interactive environment in which all
environment variables defined by the derivation are set accordingly.
The `default.nix` derivation also contains a `PATH` setting to allow
the invocation of locally installed NPM packages from the command-line.

    $ nix-shell default.nix
    # npm is now available
    $ npm --version
    6.2.0

# Step 2: Install PureScript and Pulp

Now that you are inside the Nix environment with NPM installed, you
can move on to installing PureScript and Pulp.
[Pulp](https://github.com/purescript-contrib/pulp) is essentially
a frontend for the PureScript compiler, `purs`.

    $ npm install purescript pulp
    ...
    + purescript@0.12.1
    + pulp@12.3.0
    added 133 packages in 11.614s


# Step 3: Initialize Your Project

You can opt to use `bower` or
[psc-package](https://psc-package.readthedocs.io/en/latest/index.html)
for managing dependencies
within your project. Using `psc-package` will allow you to work from
a curated set of packages for which the dependencies have been resolved.
For Haskell users, `psc-package` is more less analogous to `stack` in
the Haskell ecosystem.

If desired, you can fork the `psc-package` set yourself to guarantee
that no underlying packages versions will change behind the scenes, and
you can also use your forked repository to add custom packages you'd like
to make available in your project. Doing so is just a matter of modifying
`psc-package.json`.

    # Example of a custom package set.

    {
        "name": "my-project",
        "set": "psc-0.10.2",
        "source": "https://github.com/joeuser/package-sets.git",
        "depends": [
            "prelude"
        ]
    }

The `psc-package` binary is provided by the `default.nix` derivation that
we shelled into earlier. You can use `psc-package` to initialize your
project and install whatever packages you need. Any packages installed
via NPM or `psc-package` will be local to this repository, so you can
always get back to a blank slate just by removing the cloned repository
and starting over from scratch.

    # initialize psc-package project
    $ psc-package init
    # add repl support
    $ psc-package install psci-support
    # add list support
    $ psc-package install lists
    # add foldable support
    $ psc-package install foldable-traversable
    # install deps required for building a web app
    $ psc-package install effect
    $ psc-package install hedwig

Now you can test your setup via the repl.

    $ psc-package repl

    PSCi, version 0.12.1
    Type :? for help

    > import Prelude
    > import Data.List
    > import Data.Foldable
    > foldr (+) 0 (1..10)
    55

It's worth noting that you can side-step needing both NPM
and psc-package and offload all of your package management
needs to Nix; however, this increases the complexity on the
Nix side of the equation and falls outside of the scope of
this document.

That said, here are some resources if you want to explore
doing everything in Nix.

* [node2nix](https://github.com/svanderburg/node2nix)
* [Nix-ify Your Psc-Package Dependencies](https://qiita.com/kimagure/items/85a64437f9af78398638)

# Step 4: Supporting Web Libraries

The final step is to add libraries to facilitate developing
an [SPA](https://en.wikipedia.org/wiki/Single-page_application)
in PureScript. For this example, we'll use
[Hedwig](https://github.com/utkarshkukreti/purescript-hedwig),
a declarative PureScript library for building web applications,
as Hedwig follows [TEA](https://guide.elm-lang.org/architecture/),
a familiar design pattern for developing SPA; however, you should
explore the list of available libraries and choose whatever best
fits your needs.

    # install the required npm packages from package.json
    $ npm install
    # watch for changes and re-compile as needed
    $ npm start

From here, you can open `index.html`, and you should see Hedwig's
example counter application running in your browser. Pulp will recompile
your project whenever you make changes to your code, so you get hot
reloading for free!

# Step 5: Deploy!

The `package.json` file includes `uglify` and a script hook to minify
the Javascript emitted from `purs` and get it ready for production.
The `pulp` invocation inside of `package.json` includes the `--optimise`
flag which performs dead code elimination to minimize the footprint of
the Javascript that's generated.

    $ npm run dist

    > purescript-quickstart@0.1.0 dist /private/tmp/howto
    > mkdir -p dist/js && uglifyjs js/*.js -m -o dist/js/index.js && cp index.html dist/

    $ tree dist
    dist
    ├── index.html
    └── js
        └── index.js

    1 directory, 2 files

    $ ls -lh dist/js/index.js | awk '{print $4}'
    44K

The contents of the `dist` directory are ready to
[deploy](https://github.com/stojanovic/scottyjs) to your favorite CDN.

# Step 6: Learn More PureScript

Put simply, PureScript
[sucks the least](https://www.reddit.com/r/purescript/comments/6g6brx/why_purescript/dinwty2/)
out of all the FP frontend languages available. Take advantage of
some of the excellent resources available to further your understanding
of the language, and go
[build something](http://holdenlee.github.io/blog/posts/programming/purescript/snake-in-purescript.html)!

Helpful resources:

* [PureScript by Example](https://leanpub.com/purescript/read)
* [PureScript Resources](https://purescript-resources.readthedocs.io/en/latest/index.html)
