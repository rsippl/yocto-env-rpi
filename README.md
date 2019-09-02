## Docker based development environment for Yocto on Raspberry Pi

Tested on the Raspberry Pi 3 Model B

### Scripts
* `build-docker-image.sh`
    * builds a Docker image locally, based on `docker/Dockerfile`
    * run this *once*, or when you change the Dockerfile
* `start.sh`
    * starts the Docker container
    * exit with ctrl+D
      * destroys the container
      * data in directories mounted as volumes (e.g. home, workspace) will be persisted
* `clone-layers.sh`
    * clones the Yocto layers needed by your project
    * run *once* inside the Docker container
* `init-env`
    * initializes the environment variables, so you can issue commands like `bitbake` etc.
    * source it *every time* you start the container: `source ./init-env`
* `serial.sh`
    * connects to the board's serial port (via /dev/ttyUSB0)
    * requires `ENABLE_UART = "1"`
    * see https://learn.adafruit.com/adafruits-raspberry-pi-lesson-5-using-a-console-cable/connect-the-lead

### Directories
* `cache`
    * contains the locations used as Docker volume targets for the Yocto
    DL_DIR and SSTATE_DIR variables
    * set them in conf/local.conf:
    ```
        DL_DIR = /opt/yocto/cache/downloads
        SSTATE_DIR = /opt/yocto/cache/sstate
    ```
    * ignored by git
* `home`
    * Docker volume target for the home directory of the yocto user
    * allows you to add persistence for bash and git configuration, history, ssh keys etc.
    * is ignored by git, so to add something, you need to use `git add --force <file>`
* `conf`
    * create this directory to add Yocto config files
    (`build/conf/*.conf`) to source control
    * if it exists, `init-env` adds it to the workspace via symlink
* `workspace`
    * contains layers downloaded by `clone-layers.sh`, Yocto artifacts built by
    `bitbake`, (a symlink to) the configuration directory etc.
    * ignored by git
  
### Environment Setup

Make sure you have Ubuntu 18.04 or 16.04, other distros may or may not work.

Install Docker (see docs on the Docker site) and make sure you can start
containers as non-root user:

```
docker run --rm hello-world
```

Optional:
* add the layers your project depends on to `clone-layers.sh` 
  * you'll eventually have to add them to `conf/bblayers.conf`, too
* check `init-env` and make sure the environment init script is sourced in a correct way
  * see https://www.yoctoproject.org/docs/2.6.3/ref-manual/ref-manual.html#structure-core-script
  * see https://github.com/agherzan/meta-raspberrypi


Finally, proceed with the following (as a non-root user):
```
./build-docker-image.sh
./start.sh
./clone-layers.sh
. ./init-env
bitbake core-image-minimal
```

Copy `conf` from `workspace/build` to the project's root dir, in order to add `local.conf`, `bblayers.conf` etc. to the git repo.

### Working with the Docker Environment


```
./start.sh
. ./init-env
bitbake core-image-minimal
```

### Creating an SD Card Image

Creating an image using bitbake generates a compressed wic file in
workspace/build/tmp/deploy/images. For example, a
`bitbake core-image-minimal` on Raspberry Pi 3 will generate
`workspace/build/tmp/deploy/images/raspberrypi3/core-image-minimal-raspberrypi3.wic.gz`

Use balenaEtcher (https://www.balena.io/etcher) to write it to an SD card.

Connect to the board via USB, then use `serial.sh` to watch it boot.
