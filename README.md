# reproduce-aqua-3615

This repository is used to reproduce the issue. <https://github.com/aquaproj/aqua/issues/3615>
At the moment, @suzuki-shunsuke can't reproduce the issue.

We need to fix code to reproduce the issue.

## Requirements

- [aqua](https://aquaproj.github.io/docs/install)

## CI

It would be great if we can reproduce the issue by CI.

[workflow](.github/workflows/test.yaml)

## How To Reproduce

1. Set up and confirm mage and go are installed by aqua:

```console
aqua i --tags 'first'
$ go env | grep -E 'GOPATH|GOTOOL|GOVERSION'
GOPATH='/Users/(USER)/go'
GOTOOLCHAIN='auto'
GOTOOLDIR='/Users/(USER)/go/pkg/mod/golang.org/toolchain@v0.0.1-go1.24.0.darwin-arm64/pkg/tool/darwin_arm64'
GOVERSION='go1.24.0'
$ echo "mage built with: $(mage -version | grep -E 'built with: go')"
mage built with: built with: go1.23.5
```

```console
$ which -a go
/Users/(USER)/.local/share/aquaproj-aqua/pkgs/http/golang.org/dl/go1.23.5.darwin-arm64.tar.gz/go/bin/go
/Users/(USER)/.local/share/aquaproj-aqua/bin/go

$ aqua which go
/Users/(USER)/.local/share/aquaproj-aqua/pkgs/http/golang.org/dl/go1.24.0.darwin-arm64.tar.gz/go/bin/go
$ which -a mage
/Users/(USER)/.local/share/aquaproj-aqua/bin/mage
$ aqua which mage
/Users/(USER)/.local/share/aquaproj-aqua/pkgs/go_build/github.com/magefile/mage/v1.15.0/bin/mage
```

2. Run a mage task, forcing a rebuild/no caching.

```sh

$ mage -f -l
Targets:
  hello    outputs a friendly greeting.
```

## My steps to then reproduce the issue

- alter the go version after running install

```yaml
packages:
  - name: golang/go@go1.23.0
    tags: ['first']
```

```console
$ aqua i --tags 'first'
Downloading golang/go go1.23.0 100%
$ exec zsh -l
$ go env | grep -E 'GOPATH|GOTOOL|GOVERSION'
GOPATH='/Users/(USER)/go'
GOTOOLCHAIN='auto'
GOTOOLDIR='/Users/(USER)/go/pkg/mod/golang.org/toolchain@v0.0.1-go1.24.0.darwin-arm64/pkg/tool/darwin_arm64'
GOVERSION='go1.24.0'
$ echo "mage built with: $(mage -version | grep -E 'built with: go')"
mage built with: built with: go1.23.5
```

NOTE, I have go as a global tool setup with aqua, so I think it was taking precedence over the project version here.

So I ran with new vscode instance:

```console
$ which -a go
/Users/(USER)/.local/share/aquaproj-aqua/pkgs/http/golang.org/dl/go1.23.5.darwin-arm64.tar.gz/go/bin/go
/Users/(USER)/.local/share/aquaproj-aqua/bin/go

$ aqua which go
/Users/(USER)/.local/share/aquaproj-aqua/pkgs/http/golang.org/dl/go1.23.0.darwin-arm64.tar.gz/go/bin/go
$ which -a mage
/Users/(USER)/.local/share/aquaproj-aqua/bin/mage
$ aqua which mage
/Users/(USER)/.local/share/aquaproj-aqua/pkgs/go_build/github.com/magefile/mage/v1.15.0/bin/mage
```

Now I tried to remove:

```console
aqua rm -m p mage
rm /Users/(USER)/.local/share/aquaproj-aqua/bin/mage
$ which -a mage
mage not found
```

Now based on the current go version in the project I'd expect it to build mage with go 1.23.0

```console
$ aqua i --tags 'first'
Downloading golang/go go1.23.0 100%
aqua i
⠧ Downloading mage v1.15.0 (8.8 MB, 5.2 MB/s) [1s] %
```

Now, still the same.

```console
$ exec zsh -l
$ go env | grep -E 'GOPATH|GOTOOL|GOVERSION'
GOPATH='/Users/(USER)/go'
GOTOOLCHAIN='auto'
GOTOOLDIR='/Users/(USER)/go/pkg/mod/golang.org/toolchain@v0.0.1-go1.24.0.darwin-arm64/pkg/tool/darwin_arm64'
GOVERSION='go1.24.0'
$ echo "mage built with: $(mage -version | grep -E 'built with: go')"
mage built with: built with: go1.23.5
```

Now let me try with aqua exec explicitly:

```console
aqua rm -m p mage
rm /Users/(USER)/.local/share/aquaproj-aqua/bin/mage
$ which -a mage
mage not found
```

```console
$ aqua i --tags 'first'
aqua i
⠧ Downloading mage v1.15.0 (8.8 MB, 5.2 MB/s) [1s] %
```

```console
$ exec zsh -l
$ aqua exec go env | grep -E 'GOPATH|GOTOOL|GOVERSION'
GOPATH='/Users/(USER)/go'
GOTOOLCHAIN='auto'
GOTOOLDIR='/Users/(USER)/go/pkg/mod/golang.org/toolchain@v0.0.1-go1.24.0.darwin-arm64/pkg/tool/darwin_arm64'
GOVERSION='go1.24.0'
$ echo "mage built with: $(aqua exec mage -version | grep -E 'built with: go')"
mage built with: built with: go1.23.5
```

Now since I've got go 1.23.0 set as the project version, I want to make sure it's using this for my cli commands so let me try to force this:

```console
$ dyn_goroot=$(aqua exec -- go env GOROOT)
$ export GOROOT="$dyn_goroot"
$ export PATH="$GOROOT/bin:$PATH"
$ go env | grep -E 'GOPATH|GOTOOL|GOVERSION'
GOPATH='/Users/(USER)/go'
GOTOOLCHAIN='auto'
GOTOOLDIR='/Users/(USER)/go/pkg/mod/golang.org/toolchain@v0.0.1-go1.24.0.darwin-arm64/pkg/tool/darwin_arm64'
GOVERSION='go1.24.0'
$ cat go.mod | grep '^go '
go 1.24.0
```

ok, so i see i forget to change in this test back to 1.23.0, so let me try in the go module version too.

❌ `name: golang/go@go1.23.0` is producing the different go version than expected.

```shell
$ cat go.mod | grep '^go '
go 1.23.0
```

```console
$ exec zsh -l
$ aqua exec go env | grep -E 'GOPATH|GOTOOL|GOVERSION'
GOPATH='/Users/(USER)/go'
GOTOOLCHAIN='auto'
GOTOOLDIR='/Users/(USER)/go/pkg/mod/golang.org/toolchain@v0.0.1-go1.24.0.darwin-arm64/pkg/tool/darwin_arm64' # 👈👈👈👈👈👈👈👈  MISMATCH
GOVERSION='go1.23.0'  # 👈👈👈👈👈👈👈👈  MISMATCH
$ echo "mage built with: $(aqua exec mage -version | grep -E 'built with: go')"
mage built with: built with: go1.23.5
```

```console
aqua rm -m p mage
rm /Users/(USER)/.local/share/aquaproj-aqua/bin/mage
$ which -a mage
mage not found
```

```console
$ aqua i --tags 'first'
aqua i
aqua i
⠋ Downloading mage v1.15.0 (8.8 MB, 8.8 MB/s) [1s] # internal/coverage/rtcov
compile: version "go1.24.0" does not match go tool version "go1.23.5"
# internal/unsafeheader
compile: version "go1.24.0" does not match go tool version "go1.23.5"
# internal/godebugs
compile: version "go1.24.0" does not match go tool version "go1.23.5"
# internal/byteorder
compile: version "go1.24.0" does not match go tool version "go1.23.5"
# internal/goos
compile: version "go1.24.0" does not match go tool version "go1.23.5"
# internal/profilerecord
compile: version "go1.24.0" does not match go tool version "go1.23.5"
# internal/goarch
compile: version "go1.24.0" does not match go tool version "go1.23.5"
# math/bits
compile: version "go1.24.0" does not match go tool version "go1.23.5"
# unicode
compile: version "go1.24.0" does not match go tool version "go1.23.5"
# internal/asan
compile: version "go1.24.0" does not match go tool version "go1.23.5"
# internal/itoa
compile: version "go1.24.0" does not match go tool version "go1.23.5"
# cmp
compile: version "go1.24.0" does not match go tool version "go1.23.5"
# crypto/internal/fips140/alias
compile: version "go1.24.0" does not match go tool version "go1.23.5"
# encoding
compile: version "go1.24.0" does not match go tool version "go1.23.5"
# unicode/utf8
compile: version "go1.24.0" does not match go tool version "go1.23.5"
# log/internal
compile: version "go1.24.0" does not match go tool version "go1.23.5"
# internal/goexperiment
compile: version "go1.24.0" does not match go tool version "go1.23.5"
# internal/msan
compile: version "go1.24.0" does not match go tool version "go1.23.5"
# internal/platform
compile: version "go1.24.0" does not match go tool version "go1.23.5"
# unicode/utf16
compile: version "go1.24.0" does not match go tool version "go1.23.5"
⠙ Downloading mage v1.15.0 (8.8 MB, 8.0 MB/s) [1s] # internal/goversion
compile: version "go1.24.0" does not match go tool version "go1.23.5"
# internal/syslist
compile: version "go1.24.0" does not match go tool version "go1.23.5"
# crypto/internal/boring/sig
compile: version "go1.24.0" does not match go tool version "go1.23.5"
# internal/cpu
compile: version "go1.24.0" does not match go tool version "go1.23.5"
# sync/atomic
compile: version "go1.24.0" does not match go tool version "go1.23.5"
ERRO[0001] check file_src is correct                     aqua_version=2.45.0 env=darwin/arm64 error="check file_src is correct: build Go tool: build a go package: exit status 1" file_name=mage package_name=mage package_version=v1.15.0 program=aqua registry=local
ERRO[0001] install the package                           aqua_version=2.45.0 env=darwin/arm64 error="check file_src is correct" package_name=mage package_version=v1.15.0 program=aqua registry=local
FATA[0001] aqua failed                                   aqua_version=2.45.0 env=darwin/arm64 error="it failed to install some packages" program=aqua
```

Ok, finally! Trying to reproduce on more time before I go to next step of trying to force the GOROOT.

```console



```console
aqua rm -m p mage
rm /Users/(USER)/.local/share/aquaproj-aqua/bin/mage
$ which -a mage
mage not found
$ aqua i --tags 'first'
aqua i
$ echo 'before reloading env' && go env | grep -E 'GOPATH|GOTOOL|GOVERSION'
before reloading env
GOPATH='/Users/(USER)/go'
GOTOOLCHAIN='auto'
GOTOOLDIR='/Users/(USER)/go/pkg/mod/golang.org/toolchain@v0.0.1-go1.24.0.darwin-arm64/pkg/tool/darwin_arm64'
GOVERSION='go1.23.0'
$ exec zsh -l
$ go env | grep -E 'GOPATH|GOTOOL|GOVERSION'
GOPATH='/Users/(USER)/go'
GOTOOLCHAIN='auto'
GOTOOLDIR='/Users/(USER)/go/pkg/mod/golang.org/toolchain@v0.0.1-go1.24.0.darwin-arm64/pkg/tool/darwin_arm64'
GOVERSION='go1.23.0'
$ echo "mage built with: $(aqua exec mage -version | grep -E 'built with: go')"
# internal/godebugs
compile: version "go1.24.0" does not match go tool version "go1.23.5"
# internal/unsafeheader
compile: version "go1.24.0" does not match go tool version "go1.23.5"
# internal/goos
compile: version "go1.24.0" does not match go tool version "go1.23.5"
# internal/goarch
compile: version "go1.24.0" does not match go tool version "go1.23.5"
# internal/byteorder
compile: version "go1.24.0" does not match go tool version "go1.23.5"
# internal/coverage/rtcov
compile: version "go1.24.0" does not match go tool version "go1.23.5"
# math/bits
compile: version "go1.24.0" does not match go tool version "go1.23.5"
# internal/profilerecord
compile: version "go1.24.0" does not match go tool version "go1.23.5"
# unicode
compile: version "go1.24.0" does not match go tool version "go1.23.5"
# crypto/internal/fips140/alias
compile: version "go1.24.0" does not match go tool version "go1.23.5"
# unicode/utf8
compile: version "go1.24.0" does not match go tool version "go1.23.5"
# cmp
compile: version "go1.24.0" does not match go tool version "go1.23.5"
# internal/itoa
compile: version "go1.24.0" does not match go tool version "go1.23.5"
# internal/asan
compile: version "go1.24.0" does not match go tool version "go1.23.5"
# internal/msan
compile: version "go1.24.0" does not match go tool version "go1.23.5"
# encoding
compile: version "go1.24.0" does not match go tool version "go1.23.5"
# internal/goexperiment
compile: version "go1.24.0" does not match go tool version "go1.23.5"
# internal/cpu
compile: version "go1.24.0" does not match go tool version "go1.23.5"
# sync/atomic
compile: version "go1.24.0" does not match go tool version "go1.23.5"
# crypto/internal/boring/sig
compile: version "go1.24.0" does not match go tool version "go1.23.5"
# log/internal
compile: version "go1.24.0" does not match go tool version "go1.23.5"
# internal/goversion
compile: version "go1.24.0" does not match go tool version "go1.23.5"
# unicode/utf16
compile: version "go1.24.0" does not match go tool version "go1.23.5"
# internal/syslist
compile: version "go1.24.0" does not match go tool version "go1.23.5"
# internal/platform
compile: version "go1.24.0" does not match go tool version "go1.23.5"
ERRO[0000] check file_src is correct                     aqua_version=2.45.0 env=darwin/arm64 error="check file_src is correct: build Go tool: build a go package: exit status 1" exe_name=mage file_name=mage package_name=mage package_version=v1.15.0 program=aqua registry=local
FATA[0000] aqua failed                                   aqua_version=2.45.0 env=darwin/arm64 error="install the package: check file_src is correct" exe_name=mage package_name=mage package_version=v1.15.0 program=aqua
mage built with:
```

Let me try with forcing the GOROOT and see if still occurs.

```console
aqua rm -m p mage
rm /Users/(USER)/.local/share/aquaproj-aqua/bin/mage
$ which -a mage
mage not found
$ dyn_goroot=$(aqua exec -- go env GOROOT)
$ export GOROOT="$dyn_goroot"
$ export PATH="$GOROOT/bin:$PATH"
$ go env | grep -E 'GOPATH|GOTOOL|GOVERSION'
GOPATH='/Users/(USER)/go'
GOTOOLCHAIN='auto'
GOTOOLDIR='/Users/(USER)/go/pkg/mod/golang.org/toolchain@v0.0.1-go1.24.0.darwin-arm64/pkg/tool/darwin_arm64' # 👈👈👈👈👈👈👈👈  EXPECTED 1.23.0
GOVERSION='go1.24.0' # 👈👈👈👈👈👈👈👈  EXPECTED 1.23.0
```

next let's try to override the goroot and see if we can get it to match the versions.

```console
$ go_binary=$(aqua which go)
$ echo "go_binary: $go_binary"
go_binary: /Users/(USER)/.local/share/aquaproj-aqua/pkgs/http/golang.org/dl/go1.23.0.darwin-arm64.tar.gz/go/bin/go
unset GOROOT
$ dyn_goroot=$($go_binary env GOROOT)
$ go env | grep -E 'GOPATH|GOTOOL|GOVERSION'
GOPATH='/Users/(USER)/go'
GOTOOLCHAIN='auto'
GOTOOLDIR='/Users/(USER)/.local/share/aquaproj-aqua/pkgs/http/golang.org/dl/go1.23.0.darwin-arm64.tar.gz/go/pkg/tool/darwin_arm64'
GOVERSION='go1.23.0'
$ cat go.mod | grep '^go '
go 1.23.0
$ aqua i --tags 'first'
aqua i
$ echo 'before reloading env' && go env | grep -E 'GOPATH|GOTOOL|GOVERSION'
before reloading env
GOPATH='/Users/(USER)/go'
GOTOOLCHAIN='auto'
GOTOOLDIR='/Users/(USER)/.local/share/aquaproj-aqua/pkgs/http/golang.org/dl/go1.23.0.darwin-arm64.tar.gz/go/pkg/tool/darwin_arm64'
GOVERSION='go1.23.0'
$ exec zsh -l
$ go env | grep -E 'GOPATH|GOTOOL|GOVERSION'
GOPATH='/Users/(USER)/go'
GOTOOLCHAIN='auto'
GOTOOLDIR='/Users/(USER)/.local/share/aquaproj-aqua/pkgs/http/golang.org/dl/go1.23.0.darwin-arm64.tar.gz/go/pkg/tool/darwin_arm64'
GOVERSION='go1.23.0'
$ echo "mage built with: $(mage -version | grep -E 'built with: go')"
mage built with: built with: go1.23.5 # 👈👈👈👈👈👈👈👈  EXPECTED 1.23.0
```

Hopefully this helps, as I did manage to get this though in a rambling way. 😅

My best guess is:

- aqua doesn't seem to be taking precedence in the go version possibly from the global version, but that doesn't seem to be exclusive issue.
- When doing this process, I see mage built with `1.23.5` despite having `1.23.0` set in the project.

In terms of mage usage itself something I think this brings up as a limitation and maybe there's an approach i can use?

- Mage using Go AST to build the final binary.
- So the build version mismatching the goroot I think is causing the issue.
- Not sure, but perhaps with a new version of go updated, then mage would fail again? If so I'd have to remove mage and rebuild with the current project version.
  - I'm not certain this is happening when I do `aqua rm mage` or `aqua rm -m p mage`. Seems I have to run `rm shimpath` for it to consider a rebuild?

No rush, just a messy situation and it's caused me a lot of troubleshooting trying to figure out where the problem is.
