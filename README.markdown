# Post-build Clean Buildpack

A simple buildpack to run after all other buildpacks have completed,
which removes a set of files defined in `.slug-post-clean`, so that they
are not included in the finished slug.

## Rationale

While this may seem to duplicate functionality provided by Heroku's
`.slugignore`, there is a key difference: `.slugignore`'d files are
removed after the repo is cloned, but before any buildpack is run. They
can therefore not be involved in the build process itself.

However, it is not uncommon for there to exist files in the repo that
are necessary for the build, but are not required at runtime. There may
also be installable build dependencies that are not runtime
dependencies.

In our case, a complex front-end build involves significant CSS, JS and
image assets, along with a large installation of node modules, all of
which are used only for building the production assets, but then remain
part of the slug.

## Usage

Add the buildpack to the app:

```shell
heroku buildpacks:add https://github.com/tankfarm/heroku-buildpack-post-build-clean.git
```

The post-build-clean buildpack **must** be last in the buildpack order.

```
# .buildpacks
https://github.com/heroku/heroku-buildpack-nodejs
https://github.com/heroku/heroku-buildpack-ruby
https://github.com/tankfarm/heroku-buildpack-post-build-clean
```

The `.slug-post-clean` file supports a single declatation per line.
A declaration can include bash globs (`*` and `?` wildcards, and `[]`
classes), whitespace in the filename, comments starting with `#`.
Regular files, directories, symlinks, sockets, etc can all be removed.

```
some_huge_file.psd
some/nested/directory
why_does_this_app_even_contain_a.tiff
a file with whitespace.txt
# declaration below removes all *.log and *.out files/dirs
*.log
# declaration below removes all logfile?.txt files, except logfile0.txt
logfile[^0].txt
```
