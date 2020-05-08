# Reliable Blast UDP (CMake Enabled)

A CMake-enabled version of the Reliable Blast UDP 1.0 codebase.

https://www.evl.uic.edu/cavern/RBUDP/Reliable%20Blast%20UDP.html

## Single File Transfer

### To Transfer a file between two end-nodes

The following step are needed to transfer a file between two end-nodes

#### On the Sender side

Execute the `sendfile` on the node where the data file is present.

Typical command to execute is:

```
$ sendfile <receiver> <sending rate (Kbps)> <MTU>
```

where:

`receiver` is the IP address of the receiver.

`sending rate in Kbps` eg. 700000 to get 700Mbps network transfer or memory to memory speeds. Disk speeds are typically slower than network speeds.

`MTU` if 1500 MTU then you can specify 1460 for transport protocol if MTU == 9000, we typically use 8192 for efficient memory/page usage.

#### On the Receiver Side

Execute the `recvfile` on the receiver.

Typical command to execute is:

```
$ recvfile <sender> <path to the orig file> <path to the dest file name> <MTU>
```

where:

`sender` is the IP address of the sender.

`path to the orig file>` is the complete path to the file on the sender.

`path to the dest file name` is the complete path of the file on the receiver.

`MTU` if 1500 MTU then you can specify 1460 for transport protocol if MTU == 9000, we typically use 8192 for efficient memory/page usage.


### NOTES

1. The port being used is 38000. You can modify this in sendfile.cxx and recvfile.cxx if you have firewall issues.

2. The MTU sizes on the sender and receiver that is specified on the sender and receiver side must be identical. It crashes as the check is not being made. In future the smaller of the two will be chosen as the consensus value.

## Multiple File Transfer

### To Transfer a list of files between two end-nodes

The following step are needed to transfer a list(set) of files between two end-nodes.

#### On the Sender side

Execute the `sendfilelist` on the node where the data files are present.

Typical command to execute is:

```
$ sendfilelist <receiver> <sending rate (Kbps)> <port> <MTU>
```

where:

`receiver` is the IP address of the receiver.

`sending rate in Kbps` eg. 700000 to get 700Mbps network transfer or memory to memory speeds. Disk speeds are typically slower than network speeds.

`port` i.e the port where the server will be initialized eg. 7000.

`MTU` if 1500 MTU then you can specify 1460 for transport protocol if MTU == 9000, we typically use 8192 for efficient memory/page usage.

#### On the Receiver Side

Execute the `recvfilelist` on the receiver.

Typical command to execute is:

```
$ recvfilelist <sender> <filelist> <port> <MTU>
```

where:

`sender` is the IP address of the sender.

`filelist` is a text file that contains the list of file on the sender side and the path of the destination file on the receiver. The separator is a blank space. Details in the NOTES section below.

<port> i.e the port where the server is running eg.if the server is running on the sender at 7000, the value would be 7000.

`MTU` if 1500 MTU then you can specify 1460 for transport protocol if MTU == 9000, we typically use 8192 for efficient memory/page usage.

### NOTES

1. The MTU sizes on the sender and receiver that is specified on the sender and receiver side must be identical. It crashes as the check is not being made. In future the smaller of the two will be chosen as the consensus value.

2. The format is:

```
<complete path of file on sender> <complete path of file of receiver>
```

Check `filelist.txt` for a sample file

## Streaming

Changes contributed by slevy@ncsa.uiuc.edu:

The file(list) apps were exchanging long long file-size values, but using `htonl()`/`ntohl()` to convert them.  I think this doesn't work, so added `htonll()`/`ntohll()` functions.

Also checked sanity of packet-sequence values in each data packet. If they're out of range, but byte-swapping makes them reasonable, then do so for that and all following packets.  This autodetects having two ends with different byte order.  But we shouldn't have to do it this way; better to have an explicit negotiation, or a network-byte-order convention or something.

The file/filelist apps are still awkward though -- they map in the entire file at once, so they fail on files too large
to fit in virtual memory.  Shipping large numbers of small files works, but very inefficiently, since we pay the transfer-startup cost on every single one.

So added new methods `sendstream()` and `getstream()`, which support new apps `sendstream` and `recvstream`, intended for shipping arbitrary-length byte streams.

Typical use (see their usage messages) might be:

```
$ tar cf - somedir | sendstream <receiverhost> 600m 1460
$ recvstream <senderhost> 1460 | tar xvpf -
```

Changed a bunch of library `printf`s to `fprintf(stderr)` to avoid cluttering stdout so `recvstream` could work. 

Instantiated each of the `static const int` in the `QUANTAnet_rbudpBase_c.cxx` file; otherwise *some* linkers
(Altix ia64 at least) call them undefined at link time. (How can a compile-time constant be undefined at link time??)

Set the socket-buffer size to 8MB by default, change-able by calling `QUANTAnet_rbudpBase_c::setUDPBufSize()` before starting
to send/receive.

Adjustable verbose-ness from 0 (quiet, only serious errors shown), 1 (packet-loss stats), to 2 (as verbose as ever). New `setverbose()` method call sets it, and `-q`/`-v` options in `usend`/`urecv`/`sendstream`/`recvstream` apps use it. The apps' default (neither `-q` nor `-v`) is 1.

Disabled writing `progress.log` file unless verboseness is 2.

Add block-size (default -b 64M) and port-number (default -p 38000) options to send/recvstream.

Rearranged the parameter order on usend/urecv to make a little more sense.

Made helpful Usage messages on sendstream/recvstream and usend/urecv.

Packet-loss stats messages include transfer size and performance estimates.

_Stuart Levy, slevy@ncsa.uiuc.edu, July - Dec 2006_
