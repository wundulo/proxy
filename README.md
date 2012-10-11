proxy
=====

Java multithreaded proxy server that can handle simple GET and POST requests. This proxy also strips out user-agent and proxy-connection header information to browse in incognito mode.

Tested sites:
http://www.nytimes.com/
http://www.youtube.com/
http://www.google.com

Usage:
$ java -cp proxy.jar ProxyServer