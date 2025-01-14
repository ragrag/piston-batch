<h1 align="center">
    <a href="https://github.com/engineer-man/piston"><img src="var/docs/images/icon_circle.svg" width="25" height="25" alt="engineer-man piston"></a>
  Piston Batch
</h1>

<h3 align="center">A fork from Piston, a high performance general purpose code execution engine that supports batch submissions and code evaluation.</h3>
<br>

<p align="center">
    <a href="https://github.com/engineer-man/piston/commits/master">
    <img src="https://img.shields.io/github/last-commit/engineer-man/piston.svg?style=for-the-badge&logo=github&logoColor=white"
         alt="GitHub last commit">
    <a href="https://github.com/engineer-man/piston/issues">
    <img src="https://img.shields.io/github/issues/engineer-man/piston.svg?style=for-the-badge&logo=github&logoColor=white"
         alt="GitHub issues">
    <a href="https://github.com/engineer-man/piston/pulls">
    <img src="https://img.shields.io/github/issues-pr-raw/engineer-man/piston.svg?style=for-the-badge&logo=github&logoColor=white"
         alt="GitHub pull requests">
</p>

---

<h4 align="center">
  <a href="#About">About</a> •
  <a href="#Public-API">Public API</a> •
  <a href="#Getting-Started">Getting Started</a> •
  <a href="#Usage">Usage</a> •
  <a href="#Supported-Languages">Supported Languages</a> •
  <a href="#Principle-of-Operation">Principles</a> •
  <a href="#Security">Security</a> •
  <a href="#License">License</a>
</h4>

---
<br>

# About

<h4>
Piston is a high performance general purpose code execution engine. It excels at running untrusted and
possibly malicious code without fear from any harmful effects.
</h4>
<br>

It's used in numerous places including:
* [EMKC Challenges](https://emkc.org/challenges),
* [EMKC Weekly Contests](https://emkc.org/contests),
* [Engineer Man Discord Server](https://discord.gg/engineerman),
* [I Run Code (Discord Bot)](https://github.com/engineer-man/piston-bot) bot as well as 1300+ other servers
and 100+ direct integrations.

To get it in your own server, go here: https://emkc.org/run.

<br>

# Public API

- Requires no installation and you can use it immediately.
- Reference the Versions/Execute sections below to learn about the request and response formats.

<br>

When using the public Piston API, use the base URL:

```
https://emkc.org/api/v1/piston
```

#### GET
```
https://emkc.org/api/v1/piston/versions
```
#### POST
```
https://emkc.org/api/v1/piston/execute
```

> Important Note: The Piston API is rate limited to 5 requests per second. If you have a need for more requests than that
and it's for a good cause, please reach out to me (EngineerMan#0001) on [Discord](https://discord.gg/engineerman)
so we can discuss potentially getting you an unlimited key.

<br>

# Getting Started

## All In One

### Host System Package Dependencies

- Docker
- Docker Compose
- Node JS

### After system dependencies are installed, clone this repository:

```sh
# clone and enter repo
git clone https://github.com/engineer-man/piston
```

### Installation

```sh
docker-compose up -d piston_api
# Start the API container

cd cli && npm i && cd -
# Install all the dependencies for the cli
```

## Just Piston (no CLI)

### Host System Package Dependencies

- Docker

### Installation

```sh
echo "$GITHUB_TOKEN" | docker login https://docker.pkg.github.com -u "$GITHUB_USERNAME" --password-stdin
# Change out the $GITHUB_TOKEN and $GITHUB_USERNAME with appropritate values

docker run -v $PWD:'/piston' --tmpfs /piston/jobs -dit -p 2000:2000 --name piston_api docker.pkg.github.com/engineer-man/piston/api:latest
```

<br>

# Usage

### CLI

The CLI is the main tool used for installing packages within piston, but also supports running code.

You can execute the cli with `cli/index.js`.

```sh
# List all available packages
cli/index.js ppman list

# Install python 3.9.1
cli/index.js ppman install python 3.9.1

# Run a python script
echo 'print("Hello world!")' > test.py
cli/index.js run python 3.9.1 test.py

# Run the script using the latest version
cli/index.js run python '*' test.py

# Run using python 3.x
cli/index.js run python 3.x test.py

```

If you are operating on a remote machine, add the `-u` flag like so:

```sh
cli/index.js -u http://piston.server:2000 ppman list
```

### API

The container exposes an API on port 2000 by default.
This is used by the CLI to carry out running jobs and package managment.

#### Runtimes Endpoint
`GET /runtimes`
This endpoint will return the supported languages along with the current version and aliases. To execute
code for a particular language using the `/jobs` endpoint, either the name or one of the aliases must
be provided, along with the version.
Multiple versions of the same language may be present at the same time, and may be selected when running a job.
```json
HTTP/1.1 200 OK
Content-Type: application/json

[
  {
    "language": "bash",
    "version": "5.1.0",
    "aliases": [
      "sh"
    ]
  },
  {
    "language": "brainfuck",
    "version": "2.7.3",
    "aliases": [
      "bf"
    ]
  },
  ...
]
```

#### Execute Endpoint
`POST /jobs`
This endpoint requests execution of some arbitrary code.
- `language` (**required**) The language to use for execution, must be a string and must be installed.
- `version` (**required**) The version of the language to use for execution, must be a string containing a SemVer selector for the version or the specific version number to use.
- `files` (**required**) An array of files containing code or other data that should be used for execution.
- `files[].name` (**required**) The name of the file to upload, must be a string containing no path.
- `files[].content` (**required**) The content of the files to upload, must be a string containing text to write.
- `main` (**required**) The name of one of the files provided that should be considered the main source file which will be used as the entrypoint, must be a string and be the name of a file in `files`.
- `stdin` (**required**) The text to pass as stdin to the program. Must be a string, can be left blank.
- `args` (**required**) The arguments to pass to the program. Must be an array.
- `compile_timeout` (**required**) The maximum time allowed for the compile stage to finish before bailing out in milliseconds. Must be a number.
- `run_timeout` (**required**) The maximum time allowed for the run stage to finish before bailing out in milliseconds. Must be a number.

```json
{
    "language": "js",
    "version": "15.10.0",
    "files":[
        {
            "name": "my_cool_code.js",
            "content": "console.log(process.argv)"
        }
    ],
    "main": "my_cool_code.js",
    "stdin": "",
    "args": [
        "1",
        "2",
        "3"
    ],
    "compile_timeout": 10000,
    "run_timeout": 3000
}
```
A typical response upon successful execution will contain 1 or 2 keys `run` and `compile`.
`compile` will only be present if the language requested requires a compile stage.

Each of these keys has an identical structure, containing both a `stdout` and `stderr` key, which is a string containing the text outputted during the stage into each buffer.
It also contains the `code` and `signal` which was returned from each process.
```json
HTTP/1.1 200 OK
Content-Type: application/json

{
  "run": {
    "stdout": "[\n  '/piston/packages/node/15.10.0/bin/node',\n  '/piston/jobs/9501b09d-0105-496b-b61a-e5148cf66384/my_cool_code.js',\n  '1',\n  '2',\n  '3'\n]\n",
    "stderr": "",
    "code": 0,
    "signal": null
  }
}
```
If a problem exists with the request, a `400` status code is returned and the reason in the `message` key.
```json
HTTP/1.1 400 Bad Request
Content-Type: application/json

{
    "message": "html-5.0.0 runtime is unknown"
}
```

<br>

# Supported Languages
`bash`,
`brainfuck`,
`clojure`,
`coffeescript`,
`cow`,
`crystal`,
`dart`,
`dash`,
`deno`,
`dotnet`,
`elixir`,
`emacs`,
`erlang`,
`gawk`,
`gcc`,
`go`,
`groovy`,
`haskell`,
`java`,
`jelly`,
`julia`,
`kotlin`,
`lisp`,
`lolcode`,
`lua`,
`mono`,
`nasm`,
`nim`,
`node`,
`ocaml`,
`osabie`,
`paradoc`,
`pascal`,
`perl`,
`php`,
`prolog`,
`pure`,
`python`,
`rockstar`,
`ruby`,
`rust`,
`scala`,
`swift`,
`typescript`,
`vlang`,
`zig`,

<br>

# Principle of Operation

Piston uses Docker as the primary mechanism for sandboxing. There is an API within the container written in Node
which takes in execution requests and executees them within the container safely.
High level, the API writes any source code to a temporary directory in `/piston/jobs`.
The source file is either ran or compiled and ran (in the case of languages like c, c++, c#, go, etc.).

<br>

# Security
Docker provides a great deal of security out of the box in that it's separate from the system.
Piston takes additional steps to make it resistant to
various privilege escalation, denial-of-service, and resource saturation threats. These steps include:
- Disabling outgoing network interaction
- Capping max processes at 256 by default (resists `:(){ :|: &}:;`, `while True: os.fork()`, etc.)
- Capping max files at 2048 (resists various file based attacks)
- Cleaning up all temp space after each execution (resists out of drive space attacks)
- Running as a variety of unprivileged users
- Capping runtime execution at 3 seconds
- Capping stdout to 65536 characters (resists yes/no bombs and runaway output)
- SIGKILLing misbehaving code

<br>

# License
Piston is licensed under the MIT license.
