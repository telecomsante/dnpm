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

## Help

`dnpm` provides its own help page:

```bash
dnpm -h
```

[1]: https://www.npmjs.com/
[2]: https://www.docker.com/
