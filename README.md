# gotip

> [!WARNING]  
> This repository is experimental.

## Purpose

Sometimes you want to try out a new feature of Go, or make sure that your code works with the latest version of Go.
To do so, you can use [the `gotip` command](https://pkg.go.dev/golang.org/dl/gotip), and execute `gotip download` to 
download and build Go from the development tree.

However, this may require some time (3+ minutes in Ubuntu runners, 6+ minutes in Windows runners), so the purpose of this
repository is to have pre-compiled binaries of the latest version of Go, so you can download them and use them in your
pipelines, which usually takes few seconds (less than a minute), without having to wait for the compilation process

By default, it builds a new release [every 6 hours](.github/workflows/release.yml#L6), but you can also trigger a new build
manually.

## Usage

*You can see a full example in [.github/workflows/test.yml](.github/workflows/test.yml).*

But here are the main steps:

```yml
# .github/workflows/test.yml

...

jobs:
  test:
    strategy:
      fail-fast: true
      matrix:
        platform: [ ubuntu-latest, windows-latest, windows-2019, macos-latest ]
    runs-on: ${{ matrix.platform }}
    steps:
      - name: Download Go tip
        env:
          GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
        run: |
          gh release download ${{ matrix.platform }} --repo grafana/gotip --pattern 'go.zip'
      - name: Install Go tip
        run: |
          set -x
          unzip go.zip -d $HOME/sdk
          echo "GOROOT=$HOME/sdk/gotip" >> "$GITHUB_ENV"
          echo "GOPATH=$HOME/go" >> "$GITHUB_ENV"
          echo "$HOME/go/bin" >> "$GITHUB_PATH"
          echo "$HOME/sdk/gotip/bin" >> "$GITHUB_PATH"
      - name: Use Go
        run: |
          go version
          go test ./...
```