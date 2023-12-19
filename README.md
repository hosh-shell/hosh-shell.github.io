# Hosh

## Features

[Hosh](https://github.com/hosh-shell/hosh) is an experimental shell, featuring:

- **portability**¹
    - written in Java 11, distributed as [Uber-JAR](https://imagej.net/Uber-JAR)
    - works out-of-the-box in Windows, MacOS, Linux
    - *limited* HTTP 1.1/2.0 client (`http`) and ifconfig clone (`network`) as built-in commands
- **usability as first class citizen**²
    - interactive output displayed as table by default
    - sorting with [natural sort](https://en.wikipedia.org/wiki/Natural_sort_order)
    - ANSI colors by default
    - errors (i.e. stderr) always colored in red
    - file sizes reported by default as KB, MB, GB, ...
    - better history by default
       - record timestamps for each command (see `history` command)
       - ignoring duplicated by default (like `HISTCONTROL=ignoredups` in bash)
       - append to history is incremental and shared between all sessions
       - no limits
- **robust scripts by default**
    - as if running bash scripts with `set -euo pipefail` ([unofficial-strict-mode](http://redsymbol.net/articles/unofficial-bash-strict-mode/))
    - strongly typed (but not statically typed)
       - every value has a type *not everything is a string*
       - avoiding the need to dump/parse fields
       - basic types are: `text`, `number`, `path`, `duration` and `instant`
- **pipelines** built around schema-less records:
    - built-in commands produce *records with well-defined keys*
    - use `| schema` to inspect available keys
    - interoperability with external commands is achieved by using *single-key record* (with key `text`)
- **wrapping commands**, with before/after behavior:
    - `withTime { lines very-big-file.txt | count }` like `time command` in bash
    - `withLock file.lock { command }` run `command` as critical section guarded by `file.lock`
    - `benchmark 10 { command }` run `command` 10 times and then report best/worst/average execution time
- **built with modern tooling and concepts**
    - designed to be compatible with *Java Platform Module System* (i.e. Jigsaw)
      designed to be compatible with [Project Loom](https://wiki.openjdk.java.net/display/loom/Main)
    - *Fitness Functions* from *Evolutionary Architecture* ISBN-13: 978-1491986363)
    - *Commands run in isolated thread and communicate only via records (immutable message)*,

¹ it is not intended to conform to IEEE POSIX P1003.2/ISO 9945.2 Shell and Tools standard

² much more design and work is needed in this area

## Downloads

Binary releases:
 - [v0.1.4 - 2021-08-07](https://github.com/hosh-shell/hosh/releases/download/v0.1.4/hosh-0.1.4.jar) \[[GPG signature](https://github.com/hosh-shell/hosh/releases/download/v0.1.4/hosh-0.1.4.jar.asc)\]
 - [v0.1.3 - 2020-11-03](https://github.com/hosh-shell/hosh/releases/download/v0.1.3/hosh-0.1.3.jar) \[[GPG signature](https://github.com/hosh-shell/hosh/releases/download/v0.1.3/hosh-0.1.3.jar.asc)\]
 - [v0.1.2 - 2020-07-28](https://github.com/hosh-shell/hosh/releases/download/v0.1.2/hosh-0.1.2.jar) \[[GPG signature](https://github.com/hosh-shell/hosh/releases/download/v0.1.2/hosh-0.1.2.jar.asc)\]
 - [v0.1.1 - 2020-06-27](https://github.com/hosh-shell/hosh/releases/download/v0.1.1/hosh-0.1.1.jar) \[[GPG signature](https://github.com/hosh-shell/hosh/releases/download/v0.1.1/hosh-0.1.1.jar.asc)\]

## Getting started

Requirements: JDK11+

```
$ java -jar hosh-0.1.4.jar
hosh> echo "hello world!"
hello world!
hosh>
```

## Examples

### Sorting

Sorting is always performed using a well-defined key:
```
hosh> ls
# unsorted, following local file-system order
...
hosh> ls | schema
path size
hosh> ls | sort size
# files sorted by size
...
hosh> ls | sort path
# files sorted by alphanum algorithm
...
```

### Find top-n files by size

Walk is able to recursively walk a directory and its subdirectories, providing
file name and size:
```
hosh> walk . | schema
path size
...
```

By sorting the output of `walk` it is trivial to detect the biggest files:
```
hosh> walk . | sort size desc | take 5
aaa 2,5MB
bbb 1MB
ccc 1MB
ddd 1MB
eee 1MB
```


### HTTP

Stream line by line a TSV file via HTTPS, take first 10 lines, split each line by tab yielding a 1-indexed record and finally show a subset of keys.

Bash + wget + awk:

```
bash$ wget -q -O - -- https://git.io/v9MjZ | head -n 10 | awk -v OFS='\t' '{print $10, $1, $12}'
```

Hosh (no external commands):

```
hosh> http https://git.io/v9MjZ | take 10 | split text '\\t' | select 10 1 12
```

### Glob expansion and lambda blocks

To recursively remove all `.class` files in `target`:

`hosh> walk target/ | glob '*.class' | { path -> rm ${path}; echo removed ${path} }`

`{ path -> ... }` is lambda syntax, inside this scope is possible to use `${path}`.

### Parsing

It is possible to create records by using `regex` built-in with capturing groups:

```
hosh> git config -l | schema
text
...
hosh> git config -l | regex text '(?<key>.+)=(?<value>.+)' | take 2
key                value
credential.helper  osxkeychain
user.name          Davide Angelocola
hosh> hosh> git config -l | regex text '(?<key>.+)=(?<value>.+)' | take 2 | schema
key value
key value
```

## Inspired by

- [Collection Pipeline](https://www.martinfowler.com/articles/collection-pipeline/)
- [Bash Pitfalls](https://mywiki.wooledge.org/BashPitfalls)
- [PowerShell](https://docs.microsoft.com/en-us/powershell/)
- [Erlang](https://www.rabbitmq.com/resources/armstrong.pdf)
- [KScript (Kotlin library)](https://github.com/holgerbrandl/kscript)
- [Ammonite (Scala)](https://ammonite.io)
- [Script (Go library)](https://github.com/bitfield/script)

And some nice UI features from:
- [Nushell (Rust)](https://github.com/nushell/nushell)
- [Elvish (Go)](https://elv.sh)
- [Fish (C++)](https://fishshell.com)

## License

[MIT License](LICENSE.md)

## Sponsors

[![JetBrains](https://raw.githubusercontent.com/JetBrains/logos/master/web/jetbrains/jetbrains-variant-2.svg)](https://www.jetbrains.com/?from=hosh)
[![YourKit](https://www.yourkit.com/images/yklogo.png)](https://www.yourkit.com/java/profiler?from=hosh)

