---
layout: post
title: Diagnosing post errors with netcat

---
#Diagnosing post errors with netcat

We were trying to interact with one of Google's many APIs. We had a working command line example using curl -d to post off some form parameters. How hard could it be to port that example to clj-http? Well, it took us ages. Somehow all we could get was a 400 "Bad Request" error. I was just about to spin up another app which would print out details of the request, when our friendly neighbourhood devops, Nathan Fisherleaned over from his desk, and asked "Why not just use netcat to diagnose the issue?"

Now I might have lived a sheltered life, but I had never heard of netcat (mapcat on the other hand...) From the man pages, I got the following description:

>The nc (or netcat) utility is used for just about anything under the sun involving TCP or UDP.  It can open TCP connections, send UDP packets,
>listen on arbitrary TCP and UDP ports, do port scanning, and deal with both IPv4 and IPv6.  Unlike telnet(1), nc scripts nicely, and separates
>error messages onto standard error instead of sending them to standard output, as telnet(1) does with some.
     
It turns out by running the following at the command line:

`nc -v -l localhost 8080`

And pointing our curl at localhost --port 8080 we could see exactly what data was being sent. The -l option tells netstat to listen on a port, and -v tells it to be verbose and printout out all the data being received.

We then did the same for our clj-http project, and, of course, we weren't setting the "content-type: application/x-www-form-urlencoded" header, which was populated automatically by curl. Problem solved