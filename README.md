[![Go Reference](https://pkg.go.dev/badge/github.com/bitfield/script.svg)](https://pkg.go.dev/github.com/bitfield/script)
[![Go Report Card](https://goreportcard.com/badge/github.com/bitfield/script)](https://goreportcard.com/report/github.com/bitfield/script)
[![Mentioned in Awesome Go](https://awesome.re/mentioned-badge-flat.svg)](https://github.com/avelino/awesome-go)
![CI](https://github.com/bitfield/script/actions/workflows/ci.yml/badge.svg)
![Audit](https://github.com/bitfield/script/actions/workflows/audit.yml/badge.svg)

```go
import "github.com/bitfield/script"
```

[![Magical gopher logo](img/magic.png)](https://bitfieldconsulting.com/golang/scripting)

# What is `script`?

`script` is a Go library for doing the kind of tasks that shell scripts are good at: reading files, executing subprocesses, counting lines, matching strings, and so on.

Why shouldn't it be as easy to write system administration programs in Go as it is in a typical shell? `script` aims to make it just that easy.

Shell scripts often compose a sequence of operations on a stream of data (a _pipeline_). This is how `script` works, too.

> *This is one absolutely superb API design. Taking inspiration from shell pipes and turning it into a Go library with syntax this clean is really impressive.*\
> —[Simon Willison](https://news.ycombinator.com/item?id=30649524)

Read more: [Scripting with Go](https://bitfieldconsulting.com/golang/scripting)

# Quick start: Unix equivalents

If you're already familiar with shell scripting and the Unix toolset, here is a rough guide to the equivalent `script` operation for each listed Unix command.

| Unix / shell       | `script` equivalent |
| ------------------ | ------------------- |
| (any program name) | [`Exec`](https://pkg.go.dev/github.com/bitfield/script#Exec) |
| `[ -f FILE ]`      | [`IfExists`](https://pkg.go.dev/github.com/bitfield/script#IfExists) |
| `>`                | [`WriteFile`](https://pkg.go.dev/github.com/bitfield/script#Pipe.WriteFile) |
| `>>`               | [`AppendFile`](https://pkg.go.dev/github.com/bitfield/script#Pipe.AppendFile) |
| `$*`               | [`Args`](https://pkg.go.dev/github.com/bitfield/script#Args) |
| `base64`           | [`DecodeBase64`](https://pkg.go.dev/github.com/bitfield/script#Pipe.DecodeBase64) / [`EncodeBase64`](https://pkg.go.dev/github.com/bitfield/script#Pipe.EncodeBase64) |
| `basename`         | [`Basename`](https://pkg.go.dev/github.com/bitfield/script#Pipe.Basename) |
| `cat`              | [`File`](https://pkg.go.dev/github.com/bitfield/script#File) / [`Concat`](https://pkg.go.dev/github.com/bitfield/script#Pipe.Concat) |
| `curl`             | [`Do`](https://pkg.go.dev/github.com/bitfield/script#Pipe.Do) / [`Get`](https://pkg.go.dev/github.com/bitfield/script#Pipe.Get) / [`Post`](https://pkg.go.dev/github.com/bitfield/script#Pipe.Post) |
| `cut`              | [`Column`](https://pkg.go.dev/github.com/bitfield/script#Pipe.Column) |
| `dirname`          | [`Dirname`](https://pkg.go.dev/github.com/bitfield/script#Pipe.Dirname) |
| `echo`             | [`Echo`](https://pkg.go.dev/github.com/bitfield/script#Echo) |
| `find`             | [`FindFiles`](https://pkg.go.dev/github.com/bitfield/script#FindFiles) |
| `grep`             | [`Match`](https://pkg.go.dev/github.com/bitfield/script#Pipe.Match) / [`MatchRegexp`](https://pkg.go.dev/github.com/bitfield/script#Pipe.MatchRegexp) |
| `grep -v`          | [`Reject`](https://pkg.go.dev/github.com/bitfield/script#Pipe.Reject) / [`RejectRegexp`](https://pkg.go.dev/github.com/bitfield/script#Pipe.RejectRegexp) |
| `head`             | [`First`](https://pkg.go.dev/github.com/bitfield/script#Pipe.First) |
| `jq`     | [`JQ`](https://pkg.go.dev/github.com/bitfield/script#Pipe.JQ) |
| `ls`               | [`ListFiles`](https://pkg.go.dev/github.com/bitfield/script#ListFiles) |
| `sed`              | [`Replace`](https://pkg.go.dev/github.com/bitfield/script#Pipe.Replace) / [`ReplaceRegexp`](https://pkg.go.dev/github.com/bitfield/script#Pipe.ReplaceRegexp) |
| `sha256sum`        | [`Hash`](https://pkg.go.dev/github.com/bitfield/script#Pipe.Hash) / [`HashSums`](https://pkg.go.dev/github.com/bitfield/script#Pipe.HashSums) |
| `tail`             | [`Last`](https://pkg.go.dev/github.com/bitfield/script#Pipe.Last) |
| `tee`              | [`Tee`](https://pkg.go.dev/github.com/bitfield/script#Pipe.Tee) |
| `uniq -c`          | [`Freq`](https://pkg.go.dev/github.com/bitfield/script#Pipe.Freq) |
| `wc -l`            | [`CountLines`](https://pkg.go.dev/github.com/bitfield/script#Pipe.CountLines) |
| `xargs`            | [`ExecForEach`](https://pkg.go.dev/github.com/bitfield/script#Pipe.ExecForEach) |

# Some examples

Let's see some simple examples. Suppose you want to read the contents of a file as a string:

```go
contents, err := script.File("test.txt").String()
```

That looks straightforward enough, but suppose you now want to count the lines in that file.

```go
numLines, err := script.File("test.txt").CountLines()
```

For something a bit more challenging, let's try counting the number of lines in the file that match the string `Error`:

```go
numErrors, err := script.File("test.txt").Match("Error").CountLines()
```

But what if, instead of reading a specific file, we want to simply pipe input into this program, and have it output only matching lines (like `grep`)?

```go
script.Stdin().Match("Error").Stdout()
```

Just for fun, let's filter all the results through some arbitrary Go function:

```go
script.Stdin().Match("Error").FilterLine(strings.ToUpper).Stdout()
```

That was almost too easy! So let's pass in a list of files on the command line, and have our program read them all in sequence and output the matching lines:

```go
script.Args().Concat().Match("Error").Stdout()
```

Maybe we're only interested in the first 10 matches. No problem:

```go
script.Args().Concat().Match("Error").First(10).Stdout()
```

What's that? You want to append that output to a file instead of printing it to the terminal? *You've got some attitude, mister*. But okay:

```go
script.Args().Concat().Match("Error").First(10).AppendFile("/var/log/errors.txt")
```

And if we'd like to send the output to the terminal *as well as* to the file, we can do that:

```go
script.Echo("data").Tee().AppendFile("data.txt")
```

We're not limited to getting data only from files or standard input. We can get it from HTTP requests too:

```go
script.Get("https://wttr.in/London?format=3").Stdout()
// Output:
// London: 🌦   +13°C
```

That's great for simple GET requests, but suppose we want to *send* some data in the body of a POST request, for example. Here's how that works:

```go
script.Echo(data).Post(URL).Stdout()
```

If we need to customise the HTTP behaviour in some way, such as using our own HTTP client, we can do that:

```go
script.NewPipe().WithHTTPClient(&http.Client{
	Timeout: 10 * time.Second,
}).Get("https://example.com").Stdout()
```

Or maybe we need to set some custom header on the request. No problem. We can just create the request in the usual way, and set it up however we want. Then we pass it to `Do`, which will actually perform the request:

```go
req, err := http.NewRequest(http.MethodGet, "http://example.com", nil)
req.Header.Add("Authorization", "Bearer "+token)
script.Do(req).Stdout()
```

The HTTP server could return some non-okay response, though; for example, “404 Not Found”. So what happens then?

In general, when any pipe stage (such as `Do`) encounters an error, it produces no output to subsequent stages. And `script` treats HTTP response status codes outside the range 200-299 as errors. So the answer for the previous example is that we just won't *see* any output from this program if the server returns an error response.

Instead, the pipe “remembers” any error that occurs, and we can retrieve it later by calling its `Error` method, or by using a *sink* method such as `String`, which returns an `error` value along with the result.

`Stdout` also returns an error, plus the number of bytes successfully written (which we don't care about for this particular case). So we can check that error, which is always a good idea in Go:

```go
_, err := script.Do(req).Stdout()
if err != nil {
	log.Fatal(err)
}
```

If, as is common, the data we get from an HTTP request is in JSON format, we can use [JQ](https://stedolan.github.io/jq/) queries to interrogate it:

```go
data, err := script.Do(req).JQ(".[0] | {message: .commit.message, name: .commit.committer.name}").String()
```

We can also run external programs and get their output:

```go
script.Exec("ping 127.0.0.1").Stdout()
```

Note that `Exec` runs the command concurrently: it doesn't wait for the command to complete before returning any output. That's good, because this `ping` command will run forever (or until we get bored).

Instead, when we read from the pipe using `Stdout`, we see each line of output as it's produced:

```
PING 127.0.0.1 (127.0.0.1): 56 data bytes
64 bytes from 127.0.0.1: icmp_seq=0 ttl=64 time=0.056 ms
64 bytes from 127.0.0.1: icmp_seq=1 ttl=64 time=0.054 ms
...
```

In the `ping` example, we knew the exact arguments we wanted to send the command, and we just needed to run it once. But what if we don't know the arguments yet? We might get them from the user, for example.

We might like to be able to run the external command repeatedly, each time passing it the next line of data from the pipe as an argument. No worries:

```go
script.Args().ExecForEach("ping -c 1 {{.}}").Stdout()
```

That `{{.}}` is standard Go template syntax; it'll substitute each line of data from the pipe into the command line before it's executed. You can write as fancy a Go template expression as you want here (but this simple example probably covers most use cases).

If there isn't a built-in operation that does what we want, we can just write our own, using `Filter`:

```go
script.Echo("hello world").Filter(func (r io.Reader, w io.Writer) error {
	n, err := io.Copy(w, r)
	fmt.Fprintf(w, "\nfiltered %d bytes\n", n)
	return err
}).Stdout()
// Output:
// hello world
// filtered 11 bytes
```

The `func` we supply to `Filter` takes just two parameters: a reader to read from, and a writer to write to. The reader reads the previous stages of the pipe, as you might expect, and anything written to the writer goes to the *next* stage of the pipe.

If our `func` returns some error, then, just as with the `Do` example, the pipe's error status is set, and subsequent stages become a no-op.

Filters run concurrently, so the pipeline can start producing output before the input has been fully read, as it did in the `ping` example. In fact, most built-in pipe methods, including `Exec`, are implemented *using* `Filter`.

If we want to scan input line by line, we could do that with a `Filter` function that creates a `bufio.Scanner` on its input, but we don't need to:

```go
script.Echo("a\nb\nc").FilterScan(func(line string, w io.Writer) {
	fmt.Fprintf(w, "scanned line: %q\n", line)
}).Stdout()
// Output:
// scanned line: "a"
// scanned line: "b"
// scanned line: "c"
```

And there's more. Much more. [Read the docs](https://pkg.go.dev/github.com/bitfield/script) for full details, and more examples.

# A realistic use case

Let's use `script` to write a program that system administrators might actually need. One thing I often find myself doing is counting the most frequent visitors to a website over a given period of time. Given an Apache log in the Common Log Format like this:

```
212.205.21.11 - - [30/Jun/2019:17:06:15 +0000] "GET / HTTP/1.1" 200 2028 "https://example.com/ "Mozilla/5.0 (Linux; Android 8.0.0; FIG-LX1 Build/HUAWEIFIG-LX1) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/64.0.3282.156 Mobile Safari/537.36"
```

we would like to extract the visitor's IP address (the first column in the logfile), and count the number of times this IP address occurs in the file. Finally, we might like to list the top 10 visitors by frequency. In a shell script we might do something like:

```sh
cut -d' ' -f 1 access.log |sort |uniq -c |sort -rn |head
```

There's a lot going on there, and it's pleasing to find that the equivalent `script` program is quite brief:

```go
package main

import (
	"github.com/bitfield/script"
)

func main() {
	script.Stdin().Column(1).Freq().First(10).Stdout()
}
```

Let's try it out with some [sample data](testdata/access.log):

```
16 176.182.2.191
 7 212.205.21.11
 1 190.253.121.1
 1 90.53.111.17
```

# A `script` “interpreter”

One of the nice things about shell scripts is that there's no build process: the script file itself is the “executable” (in fact, it's interpreted by the shell). Simon Willison (and GPT-4) contributed this elegant `script` interpreter, written in `bash`:

* [`go-script`](https://til.simonwillison.net/bash/go-script)

With `go-script`, you can run `script` one-liners directly:

```sh
cat file.txt | ./goscript.sh -c 'script.Stdin().Column(1).Freq().First(10).Stdout()'
```

or create `.goscript` files that you can run using a “shebang” line:

```sh
#!/tmp/goscript.sh
script.Stdin().Column(1).Freq().First(10).Stdout()
```

# Documentation

See [pkg.go.dev](https://pkg.go.dev/github.com/bitfield/script) for the full documentation, or read on for a summary.

[![The Power of Go: Tools cover image](img/tools.png)](https://bitfieldconsulting.com/books/tools)

The `script` package originated as an exercise in my book [The Power of Go: Tools](https://bitfieldconsulting.com/books/tools):

> *Not all software engineering is about writing applications. Developers also need tooling: programs and services to automate everyday tasks like configuring servers and containers, running builds and tests, deploying their applications, and so on. Why shouldn't we be able to use Go for that purpose, too?*
>
> *`script` is designed to make it easy to write Go programs that chain together operations into a pipeline, in the same way that shell scripts do, but with the robust type checking and error handling of a real programming language. You can use `script` to construct the sort of simple one‐off pipelines that would otherwise require the shell, or special‐purpose tools.*
>
> *So, when plain Go doesn’t provide a convenient way to solve a problem, you yourself can use it to implement a domain-specific “language” that does. In this case, we used Go to provide the language of Unix‐style pipelines. But we could have chosen any architecture we wanted to suit the problem. If Go doesn’t already provide the tool you need, use Go to build that tool, then use it.*\
> —From the book

## Sources

These are functions that create a pipe with a given contents:

| Source | Contents |
| -------- | ------------- |
| [`Args`](https://pkg.go.dev/github.com/bitfield/script#Args) | command-line arguments |
| [`Do`](https://pkg.go.dev/github.com/bitfield/script#Do) | HTTP response |
| [`Echo`](https://pkg.go.dev/github.com/bitfield/script#Echo) | a string |
| [`Exec`](https://pkg.go.dev/github.com/bitfield/script#Exec) | command output |
| [`File`](https://pkg.go.dev/github.com/bitfield/script#File) | file contents |
| [`FindFiles`](https://pkg.go.dev/github.com/bitfield/script#FindFiles) | recursive file listing |
| [`Get`](https://pkg.go.dev/github.com/bitfield/script#Get) | HTTP response |
| [`IfExists`](https://pkg.go.dev/github.com/bitfield/script#IfExists) | do something only if some file exists |
| [`ListFiles`](https://pkg.go.dev/github.com/bitfield/script#ListFiles) | file listing (including wildcards) |
| [`Post`](https://pkg.go.dev/github.com/bitfield/script#Post) | HTTP response |
| [`Slice`](https://pkg.go.dev/github.com/bitfield/script#Slice) | slice elements, one per line |
| [`Stdin`](https://pkg.go.dev/github.com/bitfield/script#Stdin) | standard input |

## Modifiers

These are methods on a pipe that change its configuration:

| Source | Modifies |
| -------- | ------------- |
| [`WithEnv`](https://pkg.go.dev/github.com/bitfield/script#Pipe.WithEnv) | environment for commands |
| [`WithError`](https://pkg.go.dev/github.com/bitfield/script#Pipe.WithError) | pipe error status |
| [`WithHTTPClient`](https://pkg.go.dev/github.com/bitfield/script#Pipe.WithHTTPClient) | client for HTTP requests |
| [`WithReader`](https://pkg.go.dev/github.com/bitfield/script#Pipe.WithReader) | pipe source |
| [`WithStderr`](https://pkg.go.dev/github.com/bitfield/script#Pipe.WithStderr) | standard error output stream for command |
| [`WithStdout`](https://pkg.go.dev/github.com/bitfield/script#Pipe.WithStdout) | standard output stream for pipe |

## Filters

Filters are methods on an existing pipe that also return a pipe, allowing you to chain filters indefinitely. The filters modify each line of their input according to the following rules:

| Filter | Results |
| -------- | ------------- |
| [`Basename`](https://pkg.go.dev/github.com/bitfield/script#Pipe.Basename) | removes leading path components from each line, leaving only the filename |
| [`Column`](https://pkg.go.dev/github.com/bitfield/script#Pipe.Column) | Nth column of input |
| [`Concat`](https://pkg.go.dev/github.com/bitfield/script#Pipe.Concat) | contents of multiple files |
| [`DecodeBase64`](https://pkg.go.dev/github.com/bitfield/script#Pipe.DecodeBase64) | input decoded from base64 |
| [`Dirname`](https://pkg.go.dev/github.com/bitfield/script#Pipe.Dirname) | removes filename from each line, leaving only leading path components |
| [`Do`](https://pkg.go.dev/github.com/bitfield/script#Pipe.Do) | response to supplied HTTP request |
| [`Echo`](https://pkg.go.dev/github.com/bitfield/script#Pipe.Echo) | all input replaced by given string |
| [`EncodeBase64`](https://pkg.go.dev/github.com/bitfield/script#Pipe.EncodeBase64) | input encoded to base64 |
| [`Exec`](https://pkg.go.dev/github.com/bitfield/script#Pipe.Exec) | filtered through external command |
| [`ExecForEach`](https://pkg.go.dev/github.com/bitfield/script#Pipe.ExecForEach) | execute given command template for each line of input |
| [`Filter`](https://pkg.go.dev/github.com/bitfield/script#Pipe.Filter) | user-supplied function filtering a reader to a writer |
| [`FilterLine`](https://pkg.go.dev/github.com/bitfield/script#Pipe.FilterLine) | user-supplied function filtering each line to a string|
| [`FilterScan`](https://pkg.go.dev/github.com/bitfield/script#Pipe.FilterScan) | user-supplied function filtering each line to a writer |
| [`First`](https://pkg.go.dev/github.com/bitfield/script#Pipe.First) | first N lines of input |
| [`Freq`](https://pkg.go.dev/github.com/bitfield/script#Pipe.Freq) | frequency count of unique input lines, most frequent first |
| [`Get`](https://pkg.go.dev/github.com/bitfield/script#Pipe.Get) | response to HTTP GET on supplied URL |
| [`HashSums`](https://pkg.go.dev/github.com/bitfield/script#Pipe.HashSums) | hashes of each listed file |
| [`Join`](https://pkg.go.dev/github.com/bitfield/script#Pipe.Join) | replace all newlines with spaces |
| [`JQ`](https://pkg.go.dev/github.com/bitfield/script#Pipe.JQ) | result of `jq` query |
| [`Last`](https://pkg.go.dev/github.com/bitfield/script#Pipe.Last) | last N lines of input|
| [`Match`](https://pkg.go.dev/github.com/bitfield/script#Pipe.Match) | lines matching given string |
| [`MatchRegexp`](https://pkg.go.dev/github.com/bitfield/script#Pipe.MatchRegexp) | lines matching given regexp |
| [`Post`](https://pkg.go.dev/github.com/bitfield/script#Pipe.Post) | response to HTTP POST on supplied URL |
| [`Reject`](https://pkg.go.dev/github.com/bitfield/script#Pipe.Reject) | lines not matching given string |
| [`RejectRegexp`](https://pkg.go.dev/github.com/bitfield/script#Pipe.RejectRegexp) | lines not matching given regexp |
| [`Replace`](https://pkg.go.dev/github.com/bitfield/script#Pipe.Replace) | matching text replaced with given string |
| [`ReplaceRegexp`](https://pkg.go.dev/github.com/bitfield/script#Pipe.ReplaceRegexp) | matching text replaced with given string |
| [`Tee`](https://pkg.go.dev/github.com/bitfield/script#Pipe.Tee) | input copied to supplied writers |

Note that filters run concurrently, rather than producing nothing until each stage has fully read its input. This is convenient for executing long-running commands, for example. If you do need to wait for the pipeline to complete, call [`Wait`](https://pkg.go.dev/github.com/bitfield/script#Pipe.Wait).

## Sinks

Sinks are methods that return some data from a pipe, ending the pipeline and extracting its full contents in a specified way:

| Sink | Destination | Results |
| ---- | ----------- | ------- |
| [`AppendFile`](https://pkg.go.dev/github.com/bitfield/script#Pipe.AppendFile) | appended to file, creating if it doesn't exist | bytes written, error |
| [`Bytes`](https://pkg.go.dev/github.com/bitfield/script#Pipe.Bytes) | | data as `[]byte`, error
| [`Hash`](https://pkg.go.dev/github.com/bitfield/script#Pipe.Hash) | | hash, error  |
| [`CountLines`](https://pkg.go.dev/github.com/bitfield/script#Pipe.CountLines) | |number of lines, error  |
| [`Read`](https://pkg.go.dev/github.com/bitfield/script#Pipe.Read) | given `[]byte` | bytes read, error  |
| [`Slice`](https://pkg.go.dev/github.com/bitfield/script#Pipe.Slice) | | data as `[]string`, error  |
| [`Stdout`](https://pkg.go.dev/github.com/bitfield/script#Pipe.Stdout) | standard output | bytes written, error  |
| [`String`](https://pkg.go.dev/github.com/bitfield/script#Pipe.String) | | data as `string`, error  |
| [`Wait`](https://pkg.go.dev/github.com/bitfield/script#Pipe.Wait) | | error  |
| [`WriteFile`](https://pkg.go.dev/github.com/bitfield/script#Pipe.WriteFile) | specified file, truncating if it exists | bytes written, error  |

# What's new

| Version | New |
| ----------- | ------- |
| 0.24.1  | [`JQ`](https://pkg.go.dev/github.com/bitfield/script#Pipe.JQ) accepts JSONLines data |
| 0.24.0  | [`Hash`](https://pkg.go.dev/github.com/bitfield/script#Pipe.Hash) |
|         | [`HashSums`](https://pkg.go.dev/github.com/bitfield/script#Pipe.HashSums) |
| 0.23.0  | [`WithEnv`](https://pkg.go.dev/github.com/bitfield/script#Pipe.WithEnv) |
|         | [`DecodeBase64`](https://pkg.go.dev/github.com/bitfield/script#Pipe.DecodeBase64) / [`EncodeBase64`](https://pkg.go.dev/github.com/bitfield/script#Pipe.EncodeBase64) |
|         | [`Wait`](https://pkg.go.dev/github.com/bitfield/script#Pipe.Wait) returns error |
| v0.22.0 | [`Tee`](https://pkg.go.dev/github.com/bitfield/script#Pipe.Tee), [`WithStderr`](https://pkg.go.dev/github.com/bitfield/script#Pipe.WithStderr) |
| v0.21.0 | HTTP support: [`Do`](https://pkg.go.dev/github.com/bitfield/script#Pipe.Do), [`Get`](https://pkg.go.dev/github.com/bitfield/script#Pipe.Get), [`Post`](https://pkg.go.dev/github.com/bitfield/script#Pipe.Post) |
| v0.20.0 | [`JQ`](https://pkg.go.dev/github.com/bitfield/script#Pipe.JQ) |

# Contributing

See the [contributor's guide](CONTRIBUTING.md) for some helpful tips if you'd like to contribute to the `script` project.

# Links

- [Scripting with Go](https://bitfieldconsulting.com/posts/scripting)
- [Code Club: Script](https://www.youtube.com/watch?v=6S5EqzVwpEg)
- [Bitfield Consulting](https://bitfieldconsulting.com/)
- [Go books by John Arundel](https://bitfieldconsulting.com/books)

<small>Gopher image by [MariaLetta](https://github.com/MariaLetta/free-gophers-pack)</small>
