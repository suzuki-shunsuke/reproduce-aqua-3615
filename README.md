# reproduce-aqua-3615

This repository is used to reproduce the issue. https://github.com/aquaproj/aqua/issues/3615
At the moment, @suzuki-shunsuke can't reproduce the issue.

We need to fix code to reproduce the issue.

## Requirements

- [aqua](https://aquaproj.github.io/docs/install)

## How To Reproduce

1. Set up and confirm mage and go are installed by aqua:

```sh
aqua i
go version
mage -version
which go
aqua which go
which mage
aqua which mage
```

```console
$ go version
go version go1.24.0 darwin/arm64

$ mage -version
Mage Build Tool 1.15.0
Build Date: 2023-05-11T15:39:32Z
Commit: 9e91a03eaa438d0d077aca5654c7757141536a60
built with: go1.20
```

2. Run a mage task:

```sh
mage -l
mage hello
```

Now it works fine.

## Expected behaviour

## Actual behaviour

## Note
