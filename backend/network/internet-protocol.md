# TCP
Transmission Control Protocol
# IP
> `IP` - Internet protocol


It has 4 layers, Application, Transport, Internet, Network (physical)

## Application
With which your browser communicate (HTTP to transfer website, SMTP for emails)
> HTTP -
>   - **H**yper**T**ext - Content you see on the screen AND it's a method of transfering between a server and client
>   - **T**ransfer
>   - **P**rotocol

> **HTTPS**, is the HTTP that uses [[ssl]], to prevent man-in-the-middle attacks.

## Transport
In this layer we send our data over internet using protocols like `TCP` or `UDP`
> `TCP` chops data into a packages and sends them over individually, by the quickest allowed routs, as well as attaching header with instructions to compose it back, and if some package gets lost, we call it again to get missing packages.
> `UDP` does the same, but it doesn't verify its integrity, which makes it faster (perfect for games)

## Internet
Assembles packages with header and attaching destination point with ip address

## Network
After that, packages are send to the physical machines using `MAC`, and converted into electrical impulses to go through serial tubes
> `MAC` like ip address, but it's a local, physical address, attachet to the machine, to route to right machine
