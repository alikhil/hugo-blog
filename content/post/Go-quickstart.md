---
title: "Go Quickstart"
date: 2018-04-06T16:22:35+03:00
coverImage: https://cdn-images-1.medium.com/max/2000/1*y4l5IIUkJj5uP9RGM8U-Fg.png # "/images/golang.png" 
coverMeta: out
coverSize: partial

thumbnailImagePosition: left
thumbnailImage: "images/gopher.png"

categories:
- programming

tags:
- Go
- vscode
- tutorial
---

Hi folks! 
It's been a long time since I have published the last post, but now I came back with short quickstart guide in **Go**.

In this tutorial, we will configure Go environment in VS Code and write our first program in Go.

<!--more-->

### Install Go

The first thing that you need to do it's to install Go on your computer. To do so, download installer for your operating system from  [here](https://golang.org/dl/) and then run the installer.


### Configure GOPATH

By language convention, Go developers store all their code in a single place called *workspace*. Go also puts dependency packages in the workspace. So, in order to Go perform correctly, we need to set `GOPATH` variable with the path to the workspace. 

#### MacOS and Linux

Set the `GOPATH` envar with workspace

```bash
export GOPATH=$HOME/go
```

Also, we need to add `GOPATH/bin` to `PATH` in order to run compiler Go programs:

```bash
export PATH=$PATH:$GOPATH/bin
```

### Configure VS Code

Install [official Go extension](https://github.com/Microsoft/vscode-go).

Install delve debugger:

```bash
go get -u github.com/derekparker/delve/cmd/dlv
```

I recommend you to add the following lines to your VS Code user settings:

{{< codeblock "settings.json" "json" >}}
{
    "go.autocompleteUnimportedPackages": true,
    "go.formatTool": "gofmt"
}
{{< /codeblock >}}


#### Windows

Create `GOPATH` envar:

```sh
set GOPATH=c:\Users\%USERNAME%\go
```

Also, we need to add `GOPATH\bin` to `PATH` in order to run compiler Go programs:

```sh
set PATH=%PATH%;%GOPATH%\bin
```

### Create project

Move to your `GOPATH/src` directory. Create a directory for your project:

```bash
cd $GOPATH/src
mkdir -p github.com/alikhil/hello-world-with-go
```

Open it using vscode:

```bash
code github.com/alikhil/hello-world-with-go
```

### Hello World!

Let's create a file named `program.go` and put the following code there:


{{< codeblock "program.go" "go" >}}

package main

import "fmt"

func main() {
    fmt.Println("¡Hola, mundo!")
}

{{< /codeblock >}}


### Run the program

Finally, to run the program by pressing the `F5` button in VS Code and you should see the message printed to *Debug Console*.

That's all! My congratulations, you have just written your first program in Go! 

### Troubleshooting

If you fail to run your program and there is some message like **"Cannot find a path to `go`"**.
Try to add to your `PATH` envar with path directory where `go` binary is stored.

For example in MacOS I have added following line to my `~/.bash_profile`:

```bash
export PATH=/usr/local/go/bin:$PATH
```