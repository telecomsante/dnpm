# dnpm

A simple bash script to ease [npm][1] usage through [docker][2].

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
dnpm -w path/to/a/node/project "apk add --no-cache git openssh" "npm install --unsafe-perm"
```

> `--unsafe-perm` was passed as an argument to [npm][1] as the default [docker][2] image used by `dnpm` is [mhart/alpine-node][4].
> The user used by this image is `root` and [npm][1] will refuse to execute external commands (like bower) as `root` by default.

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

There seems to be no way to instruct [node-gyp][5] to change its `devdir` through an [npm][1] option or an environment variable (`--nodedir` is not an option as it also impacts the way [npm][1] reaches `node`).

In order to work around this limitation `dnpm` binds temporarily `~/.node-gyp` to a temporary folder in the container.
This means that `dnpm` also ensures that the `~/.node-gyp` folder exists.
This is for now the only modification that `dnpm` might operate on the user account.

[node-gyp][5] will probably need additional tools like `python` and other build tools, so for the default image ([alpine-node][4]):

```bash
dnpm -w path/to/a/node/project "apk add --no-cache build-base python" "npm install"
```

If you think installing all these tools takes too much time, you can use your own build image but do not forget that it has to be close to your deployment image (same operating system revision for instance):

```bash
dnpm -w path/to/a/node/project -i custom_image_name "npm install"
```

> An example of an [alpine-node][4] image having all the build tools necessary for using [node-gyp][5] is [erdii/nodejs-alpine-buildtools][6].
>
> The alternate image must host a user having administration rights.
>
> Also note that operating with a user other than root almost certainly means that its home directory will not be `/root` and that you will have to use the `-h` option.

[1]: https://www.npmjs.com/
[2]: https://www.docker.com/
[3]: https://github.com/whilp/ssh-agent
[4]: https://hub.docker.com/r/mhart/alpine-node/
[5]: https://github.com/nodejs/node-gyp
[6]: https://hub.docker.com/r/erdii/nodejs-alpine-buildtools/
