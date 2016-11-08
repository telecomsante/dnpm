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

## Usage

`dnpm` can mainly be used to operate [npm][1] installs:

```bash
dnpm -p path/to/a/node/project -c "npm install"
```

For more complex situations involving bower and private git repositories, you might want to do things like:

```bash
dnpm -p path/to/a/node/project -c "apk add -U git openssh" -c "npm install --unsafe-perm"
```

> `--unsafe-perm` was passed as an argument to [npm][1] as the default [docker][2] image used by `dnpm` is [mhart/alpine-node][4].
> The user used by this image is `root` and [npm][1] will refuse to execute external commands (like bower) as `root` by default.

## Password protected SSH keys

If your npm commands involve using an SSH key protected by a password, you can start an [SSH agent container][3] and add your SSH key this way:

```bash
docker run -d -v ssh:/ssh --name=ssh-agent whilp/ssh-agent:latest
docker run --rm -v ssh:/ssh -v $HOME:$HOME -it whilp/ssh-agent:latest ssh-add $HOME/.ssh/id_rsa
```

Then simply use the `-s` option of `dnpm` to take into account the `ssh-agent`:

```bash
dnpm -p path/to/a/node/project -s ssh -c "npm install"
```

Where `ssh` (the `-s` argument) is the [docker][2] volume used by the [SSH agent container][3].

## Help

`dnpm` provides its own help page:

```bash
dnpm -h
```

[1]: https://www.npmjs.com/
[2]: https://www.docker.com/
[3]: https://github.com/whilp/ssh-agent
[4]: https://hub.docker.com/r/mhart/alpine-node/
