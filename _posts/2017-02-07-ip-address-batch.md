---
title: Getting the IP Address in Batch
date: 2017-02-07 18:40:00 +0100
categories: [Software]
tags: [batch, Windows]
---

This is a way to **get the computer's IP address** from a *Batch* script with a single line of code. Jump to [TL;DR](#tldr) for the copy-paste ready code.

So... what do we need the IP address of this computer for? Do we need the our network address or our public address?

## Getting Local IP Address

If the IP we need is the **internal address** (the one within current network's boundaries), we can ask our local DNS server (which knows our machine and our address). Local DNS server is running inside our router.

The correct way to proceed would be to run `ipconfig` command and parse it's output, then handpick from the list of results the address we actually want.

What if we want this procedure to be completely automatic? Then we have two options: ask the DNS server with `nslookup` command; or use local cache with `ping` command. What we will be doing is ping our own machine and see what Windows decided to be the best address to use.

We will use `ping` command to ping our local machine using *IPv4* and sending only one packet. Since pinging the local machine, this operation should not halt and should never fail. When offline, this command will ping *127.0.0.1*.

```text
C:\Users\User>ping -4 -n 1 %ComputerName%
Pinging MY-COMPUTER [192.168.1.139] with 32 bytes of data:
Reply from 192.168.1.139: bytes=32 time<1ms TTL=128

Ping statistics for 192.168.1.139:
    Packets: Sent = 1, Received = 1, Lost = 0 (0% loss),
Approximate round trip times in milli-seconds:
    Minimum = 0ms, Maximum = 0ms, Average = 0ms
```

Let's first check what happens if our locale is not English...

```text
C:\Users\User>ping -4 -n 1 %ComputerName%
Esecuzione di Ping MY-COMPUTER [192.168.1.139] con 32 byte di dati:
Risposta da 192.168.1.139: byte=32 durata<1ms TTL=128

Statistiche Ping per 192.168.1.139:
    Pacchetti: Trasmessi = 1, Ricevuti = 1,
    Persi = 0 (0% persi),
Tempo approssimativo percorsi andata/ritorno in millisecondi:
    Minimo = 0ms, Massimo =  0ms, Medio =  0ms
```

We can extract the line with the IP inside square brackets by piping the results into `findstr` command.

```text
C:\Users\User>ping -4 -n 1 %ComputerName% | findstr "["
Pinging MY-COMPUTER [192.168.1.139] with 32 bytes of data:
```

And finally, we can discard anything outside of the brackets.

```
C:\Users\User>for /f "delims=[] tokens=2" %a in ('ping -4 -n 1 %ComputerName% ^| findstr [') do echo %a
192.168.1.139
```

Do we need this code to be *Batch*? Let's create a *local_ip.bat* file with the following contents…

```batch
@echo off

for /f "delims=[] tokens=2" %%a in ('ping -4 -n 1 %ComputerName% ^| findstr [') do set ThisIP=%%a

echo %ThisIP%
```

And we are done. 😄

> Note: all this commands are available since *Windows XP* and we never rely on localized strings, we can thus use this script anywhere.

## Getting Public IP Address

If we need the **public address** of the outer machine in current network, then we need to ask for a little help from the outside: we have to call an external service and ask where they see our request coming from.

We well be using the free [ipify](https://www.ipify.org/) service.

```
C:\Users\User>powershell Invoke-RestMethod api.ipify.org
188.216.217.9
```

> Note: I also tried with `nslookup myip.opendns.com. resolver1.opendns.com` command but it didn't prove reliable as it failed in some of my networks.

Same as before: let's make this a Batch script. Name it *public_ip.bat* and paste the following code…

```batch
@echo off

for /f %%a in ('powershell Invoke-RestMethod api.ipify.org') do set ThisIP=%%a

echo %ThisIP%
```

> Note: unlike the script to read network IP address, this one relies on *PowerShell* which is only included since *Windows 7*. This script will thus not work on *Windows XP* and *Vista* out of the box.

Thanks for reading, I hope this guide will prove useful to you. Feel free to leave a comment in the section below! 😁

---

## TL;DR

```batch
@echo off

for /f "delims=[] tokens=2" %%a in ('ping -4 -n 1 %ComputerName% ^| findstr [') do set NetworkIP=%%a

for /f %%a in ('powershell Invoke-RestMethod api.ipify.org') do set PublicIP=%%a

echo Network IP: %NetworkIP%
echo Public IP: %PublicIP%
```

---

> References

> https://www.microsoft.com/resources/documentation/windows/xp/all/proddocs/en-us/nslookup.mspx?mfr=true

> https://www.microsoft.com/resources/documentation/windows/xp/all/proddocs/en-us/findstr.mspx?mfr=true

> https://www.microsoft.com/resources/documentation/windows/xp/all/proddocs/en-us/ping.mspx?mfr=true

> http://stackoverflow.com/q/5898763/1135019
