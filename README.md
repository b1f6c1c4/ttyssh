# ttyssh

> ssh with tty (serial port) forwarding

Tunnel your serial port over ssh!

## Usage

It's **highly recommanded** to setup `ssh` [ControlMaster](https://unix.stackexchange.com/questions/244748/how-to-properly-use-ssh-controlmaster).

**Please set baud rate locally BEFORE running `ttyssh`.**

```sh
stty -F /dev/ttyUSB0 115200
```

Now launch `ttyssh` just as how you launch `ssh`:

```sh
ttyssh -t /dev/ttyUSB0 -T '$HOME/tty' <user>@<host>
```

You should expect a remote shell, just like a regular ssh gives you.
On the remote system, you may treat $HOME/tty as a regular tty (serial port):

```sh
screen $HOME/tty
```

It also works with `pySerial`.

When the remote shell exits, the tunnel is tore down.

If you have multiple serial ports to tunnel, try:

```sh
ttyssh -t /dev/ttyUSB0 -T '$HOME/tty0' -P 23366 -p 23366 -N <user>@<host> &
ttyssh -t /dev/ttyUSB1 -T '$HOME/tty1' -P 23367 -p 23367 -N <user>@<host> &
ssh <user>@<host>
kill -s SIGINT $(jobs -p)
```

**Notice: If you `kill -s SIGKILL` or `kill -s SIGTERM` the `ttyssh` process,
the tunnel will keep running and cause big trouble. Do only if you know what
you are doing. Refer to source code for this.**

## Install

`ttyssh` requires `ssh` to be available in `PATH` on local machine.
`ttyssh` also depends on a tiny program `socat` on **both local and remote** machines.
It's shipped by some distros. You can check if you have it by `which socat`.
If you don't have one, choose one of the following:

- `pacman -S socat`
- `apt-get install socat`
- `yum install socat`
- Download [source](http://www.dest-unreach.org/socat/), untar, `./configure && make`

After you have both `ssh` and `socat` ready, download the script from this repo:

```sh
curl -fSSL https://raw.githubusercontent.com/b1f6c1c4/ttyssh/master/ttyssh > ~/.local/bin/ttyssh && chmod +x ~/.local/bin/ttyssh
```

**You only need to install `ttyssh` on your local machine.**

Using ssh ControlMaster is highly encouraged. A single `ttyssh` will execute
`ssh` three times so you may not want to authenticate three times.
See [here](https://unix.stackexchange.com/questions/244748/how-to-properly-use-ssh-controlmaster) for more information.

## Use cases

1. Useful if we want to debug serial-port devices connected to my computer
    using software installed on remote machines
2. Useful if we want to debug serial-port devices connected to remote machines
    using software installed on my computer
3. Useful if we want to debug serial-port devices connected to remote machines
    using software installed on another remote machines

## Technical details

`ttyssh` basically does three things:

1. On local machine, `socat /dev/ttyUSB0 tcp4-listen:localhost:23366`
2. On remote machine, `socat tcp4-connect:localhost:23366 $HOME/tty`
3. On local machine, `ssh -R 23366:localhost:23366` (This is ssh reverse tunnel)

Some extra code is added to ensure the links are re-established upon failure,
while respond to SIGINT on the local side.
