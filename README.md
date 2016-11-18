# dnpm

A simple bash script to ease [npm][1] usage through [docker][2].

`dnpm` is mainly of use when needing to operate preliminary [npm][1] tasks before building a [docker][2] image. This is in particular necessary when you do not want to pollute the image to build with the building environment (for instance with credentials needed to get dependencies from private repositories).
An alternative is to directly operate these tasks in the [Dockerfile][10] but this also means ensuring that nothing remains from the building environment which can sometimes be a [cumbersome task][11] even if it is quite well described in [npm documentation][12] (read the article through its end to realize that you will actually need to squash your image).

So the main point of `dnpm` is to ensure that your configuration and credentials are available within this preliminary build container.
This is provided by **read-only** mounting the host user home directory and ensuring that part of the host user environment is also available within the container.

The only host volume that is not mounted read-only is the `workdir` which is the current project directory (the one with the `package.json` file) and hence should be protected by a versioning system like [git][8].

The default image used by the `dnpm` container is [mhart/alpine-node][4].
This can be easily changed by using the `-i` option.

## Installation

```bash
curl -L -O https://raw.githubusercontent.com/telecomsante/dnpm/master/dnpm
```

> If a tagged revision of the tool is needed:
>
> ```bash
> curl -L -O https://raw.githubusercontent.com/telecomsante/dnpm/TAG/dnpm
> ```

## Help

`dnpm` provides its own help page:

```bash
dnpm --help
```

## Usage

`dnpm` can mainly be used to operate [npm][1] installs:

```bash
dnpm -w path/to/a/node/project "npm install"
```

For more complex situations involving bower and private git repositories, you might want to do things like:

```bash
dnpm -w path/to/a/node/project "apk add --no-cache git openssh" "npm install"
```

## Password protected SSH keys

If your commands involve using an SSH key protected by a password, you can start an [SSH agent container][3] and add your SSH key this way:

```bash
docker run -d -v ssh:/ssh --name=ssh-agent whilp/ssh-agent:latest
docker run --rm -v ssh:/ssh -v $HOME:$HOME -it whilp/ssh-agent:latest ssh-add $HOME/.ssh/id_rsa
```

Then simply use the `-s` option of `dnpm` to take into account the `ssh-agent`:

```bash
dnpm -w path/to/a/node/project -s ssh "npm install"
```

Where `ssh` (the `-s` argument) is the [docker][2] volume used by the [SSH agent container][3].

## When [node-gyp][5] is needed

There seems to be no way to instruct [node-gyp][5] to change its `devdir` through an [npm][1] option or an environment variable ([`--nodedir`][7] is not an option as it also impacts the way [npm][1] reaches [Node.js][9] sources and heavily depends on the docker image used).

In order to work around this limitation `dnpm` binds temporarily `~/.node-gyp` to a temporary folder in the container.
This means that `dnpm` also ensures that the `~/.node-gyp` folder exists.
This is for now the only modification that `dnpm` might operate on the host user account.

[node-gyp][5] will probably need additional tools like [python][13] and other build tools, so for the default image ([alpine-node][4]):

```bash
dnpm -w path/to/a/node/project "apk add --no-cache build-base python" "npm install"
```

If you think installing all these tools takes too much time, you can use your own build image but do not forget that it has to be close to your deployment image (same operating system and [Node.js][9] revisions for instance):

```bash
dnpm -w path/to/a/node/project -i custom_image_name "npm install"
```

> An example of an [alpine-node][4] image having all the build tools necessary for using [node-gyp][5] is [erdii/nodejs-alpine-buildtools][6].
>
> The alternate image must host a user having administration rights (without using su or sudo) as the entry point of `dnpm` needs to operate bind mounts within the container.
>
> Also note that operating with a user other than root almost certainly means that its home directory will not be `/root` and that you will have to use the `-h` option.

[1]: https://www.npmjs.com/
[2]: https://www.docker.com/
[3]: https://github.com/whilp/ssh-agent
[4]: https://hub.docker.com/r/mhart/alpine-node/
[5]: https://github.com/nodejs/node-gyp
[6]: https://hub.docker.com/r/erdii/nodejs-alpine-buildtools/
[7]: https://github.com/nodejs/node-gyp/issues/21#issuecomment-180048770
[8]: https://git-scm.com/
[9]: https://nodejs.org/
[10]: https://docs.docker.com/engine/reference/builder/
[11]: https://github.com/npm/npm/issues/7995
[12]: https://docs.npmjs.com/private-modules/docker-and-private-modules
[13]: https://www.python.org/
