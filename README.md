# ICU development with Docker

This repository creates ICU4C releases using Docker for multiple Linux versions. The binary files results may be added to [ICU releases](https://github.com/unicode-org/icu/releases) for public distribution, especially for the release candidates ("72 RC") and fully released versions (e.g., "72.1").

To use this repository you will need:

* Access to the [ICU repository in Github](https://www.github.com/unicode-org/icu)
* Know the branch / tag of interest within that repository
* Docker installed and configured

This repository contains:
* A set of docker files for the various Linux versions
* Makefile for driving the compiling, checking, and building of these versions
* Some utility scripts to finalize release files for uploading to public directories.

ICU-docker runs a *docker-compose* command for each of the Linux targets. The outputs are created in a subdirectory `./dist`.

Each *docker-compose* execution sets up `ccache` to share cached compiler output in `./tmp/.ccache` and expects an ICU source directory under `./src/icu`

## Build the distributions

The following steps create binary files for each docker file in `./dockerfiles` and also Fedora and Ubuntu binary files.

- Install [Docker](http://docker.io)
- Copy the code for a given branch of [ICU source in github](https://github.com/unicode-org/icu).

  ```
  cd src
  export BRANCH=maint/maint-72  # Set this up to use the git ref (branch / Github release tag) you need.

  git clone --branch $BRANCH --depth 1 https://github.com/unicode-org/icu.git
  cd icu
  git lfs fetch; git lfs checkout
  cd ../..
  ```
- Compile and Run tests to verify the ICU4C version
  ```
  make check-all
  ```
- Build binaries, data, and source as .zip and .tgz files in the ./dist directory
  ```
  make dist
  ```       
  Note that some filenames in ./dist/ will need to be adjusted so be sure to rename. Use the script sort-out-dist.sh, and then manually adjust as needed to match the names as in previous releases.
  
- Sort and rename files into `dist/icu4c-*/*`

Each binary needs to include the version label, e.g., "69rc" for the release candidate of ICU version 69. The general availability for that would be "69.1". 
  ```
  ./sort-out-dist.sh
  ls -l dist/icu4c-*
  ```
  **Hint:** Check the names of files in a previous release such as [Release ICU 72.1](https://github.com/unicode-org/icu/releases/tag/release-72-1) in order to check if the renaming was successful. If not, perform manual renaming as needed.

### Optional: Link `src/` to `/src` on your system

  This symlink will give access to the error messages generated inside each docker container. For example: `Error in /src/icu/somefile.cpp`.
  ```
  sudo ln -sv `pwd`/{src,dist} /
  ```

## Verify that the distributions work

Perform some command line builds to verify the release. Use `docker-compose run` with each of the releases to check the build. Do this for each of the target Linux versions in `./dockerfiles`.
  ```
  $ docker-compose run ubuntu bash
  # This creates a temporary docker shell with a name such as 'build@59b67f6c5058:~'
  build@59b67f6c5058:~$ /src/icu/icu4c/source/configure
  # This will show the ICU version number of the release just created.
  build@59b67f6c5058:~$ make check  # To run all ICU4C tests
  ...
  build@59b67f6c5058:~$ exit  # To leave the docker shell
  ```

## Customization

If desired, you can create a `Makefile.local` that can point to a different
docker-compose.yaml:

```
# Makefile.local
DOCKER_COMPOSE=docker-compose -f local-docker-compose.yml
```

## Author

Steven R. Loomis

## License

ICU license, see [LICENSE](LICENSE)
