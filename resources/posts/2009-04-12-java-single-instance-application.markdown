---
title: Java Single Instance Application
tags: java
---

Sometimes you want only one instance of your application running, at any
one time. Java does not provide any API to detect if another instance of
your application is running or not.

However there are two popular ways of forcing single instance,

 - Acquire a lock on some magic file.
 - Start listening on a socket.

Both techniques has pros and cons.


#### Socket Technique

With this technique we start listening on a port, only one process can
listen on a socket so after first instance of our application binds
itself to the socket other instances will get BindException, which means
we are already running.

        try{        
            ServerSocket socket = 
                new ServerSocket(9999, 10, InetAddress.getLocalHost());

        }catch(java.net.BindException b){
            System.out.println("Already Running...");
        }catch( Exception e ) { 
            System.out.println(e.toString());
        }

Cons of this approach is that some virus scanners will give a warning
when an application starts listening on a socket, depending on your user
base this could be interpreted badly. You should pick a port number
thats not commonly used and high or you won't even get a single instance
of your application running.

#### Lock Technique

We try to acquire a lock on a file in the applications data directory or
on the applications main class file, if we can't, then it is safe to
assume we are already running, and act accordingly.

        try{
            RandomAccessFile randomFile = 
                new RandomAccessFile("single.class","rw");

            FileChannel channel = randomFile.getChannel();

            if(channel.tryLock() == null) 
                System.out.println ("Already Running...");      
        }catch( Exception e ) { 
            System.out.println(e.toString());
        }

Downside with this approach is it prone to I/O errors. Both techniques
work and both has, ups and downs, pick the one that fits your situation.
