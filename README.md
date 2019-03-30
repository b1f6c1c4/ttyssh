# ttyssh

> ssh with tty (serial port) forwarding

Tunnel your serial port over ssh!

## Usage

**Please set baud rate locally BEFORE running `ttyssh`.**

```sh
ttyssh -t /dev/ttyUSB0 -T '$HOME/tty' <user>@<host>
# Now in the remote shell, try
# screen $HOME/tty
```

## Use cases

1. Useful if we want to debug serial-port devices connected to my computer
    using software installed on remote machines
2. Useful if we want to debug serial-port devices connected to remote machines
    using software installed on my computer
3. Useful if we want to debug serial-port devices connected to remote machines
    using software installed on another remote machines

## Install

```sh
curl -fSSL https://raw.githubusercontent.com/b1f6c1c4/ttyssh/master/ttyssh > ~/.local/bin/ttyssh && chmod +x ~/.local/bin/ttyssh
```
