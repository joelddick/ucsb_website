# HTTP and Application Servers
.fx: title

__CS290B__

Dr. Bryce Boe

October 8, 2015

---

# Today's Agenda

* TODO
* HTTP Server Architectures
* C10K Problem
* Application Servers

---

# TODO

## Should be done

* Ruby Codecademy
* Agile Web Development with Rails Chapters 1 through 8
* Familiarity with Git (you'll get more practice)

## Before Tuesday's Class:

* [Dynamic Load Balancing on Web-server Systems](http://www.ics.uci.edu/~cs230/reading/DLB.pdf)
  by Cardellini, Colajanni, and Yu.

## By End of Today's Lab:

* Add member names and photo to project's README on github
* Create team project on pivotal tracker and add me (bryce.boe@appfolio.com) as
  me as collaborator

---

# HTTP Server Architectures

* Single Threaded (no concurrency)
* Process per request
* Process pool
* Thread per request
* Process/thread worker pool
* Event-driven

---

# Single Threaded HTTP Servers

    bind() to port 80 and listen()
    loop forever
        accept() a socket connection
        while we can read from the socket
            read() a request
            process that request
            write() its response
        close() the socket connection

> If another request comes in while we're within the loop what happens?

---

# Single Threaded Problem

If a single threaded web service does not process the request quickly, other
clients end up waiting or dropping their connections.

We are building web application not web sites. As a result:

* The requests are usually more complicated than serving a file from disk.
* It is common to have a web request doing a significant amount of computation
  and business logic.
* It is common to have a web request result in connections to multiple external
  services, e.g., databases, and caching stores
* These requests can be anything: lightweight or heavyweight, IO intensive or
  CPU intensive

We can solve these problems if the thread of control that processes the request
is separate from the thread that `listen()`s and `accept()`s new connections.

---

# Process per Request HTTP Server

Handle each request as a subprocess:

.fx: img-left

![forking web server](img/server_forking.png)

    bind() to port 80 and listen()
    loop forever
        accept() a socket connection
        if fork() == 0  # child process
            while we can read from the socket
                read() a request
                process that request
                write() its response
            close() the socket connection
            exit()

---

# Process per Request HTTP Server

## Strengths

* Simple
* Provides easy isolation between requests
* No threading issues

## Weaknesses

* Does each request duplicate the process memory?
* What happens as the CPU load increases?
* How efficient is it to fire up a process on each request?
    * How much setup and tear down work is necessary?

---

# Process Pool HTTP Server

.fx: img-left

![process pool web server](img/server_process_pool.png)

Instead of spawning a process for each request create a pool of N processes at
start-up and have them handle incoming requests when available.

The children processes `accept()` the incoming connections and use shared
memory to coordinate.

The parent process watches the load on its children and can adjust the pool
size as needed.

---

# Process Pool HTTP Server

## Strengths

* Provides easy isolation between requests
* Children can die after _M_ requests to avoid memory leakage
* Process setup and tear down costs are minimized
* More predictable behavior under high load
* No threading issues

## Weaknesses

* More complex than process per request
* Many processes can still mean a large amount of memory consumption

This web server architecture is provided by the Apache 2.x MPM "Prefork" module.

---

# Thread per Request HTTP Server

Why use multiple processes at all? Instead we can use a single process and
spawn new threads for each request.

.fx: img-left

![http server thread per request](img/server_threaded.png)

    bind() to port 80 and listen()
    loop forever
        accept() a socket connection
        pthread_create()  # function that...
            while we can read from the socket
                read() a request
                process that request
                write() its response
            close() the socket connection
            # thread dies


---

# Thread per Request HTTP Server

## Strengths

* Relatively simple
* Reduced memory footprint compared to multi-processed

## Weaknesses

* Request handling code must be thread-safe
* Pushing thread-safety to the application developer is not ideal
* Setup and tear down needs to occur for each thread (or shared data
  needs to be thread-safe)
* Memory leaks?

---

# Process/Thread Worker Pool Server

.fx: img-left

![http process/thread worker pool](img/server_worker_pool.png)

Combination of the two techniques.

Master process spawns processes, each with many threads. Master maintains
process pool.

Processes coordinate through shared memory to `accept()` requests.

Fixed threads per request, scaling is done at the process level.

---

# Process/Thread Worker Pool Server

## Strengths

* Faults isolated between processes, but not threads
* Threads reduce memory footprint
* Tunable level of isolation
* Controlling the number of processes and threads allows for predictable
  behavior under load

## Weaknesses

* Requires thread-safe code
* Uses more memory than an all-thread based approach

This web server architecture is provided by the Apache 2.x MPM "Worker" module.

---

# C10K Problem

Originally posed in 1999 by Dan Kegel.

> Given a 1 GHz machine with 2GB of RAM, and a gigabit Ethernet card, can we
> support 10,000 simultaneous connections?

## 20,000 clients means each gets:

* 50 KHz of CPU
* 100 KB of RAM
* 50 Kb/second of network

"It shouldn't take any more horsepower than that to take four kilobytes from
the disk and send them to the network once a second for each of twenty thousand
clients."

> What makes managing concurrent connections difficult?

Source: [http://www.kegel.com/c10k.html](http://www.kegel.com/c10k.html)

---

# Each Client is...

* Reading from the network socket
* Parsing its request
* Opening a file on disk
* Read the file into memory
* Write the memory to network

---

# Each Client is... __blocking__

* Reading from the network socket (__blocking__)
* Parsing its request
* Opening a file on disk  (__blocking__)
* Read the file into memory  (__blocking__)
* Write the memory to network  (__blocking__)

---

# Waiting on I/O

Every time a process is waiting on I/O it is not runnable, and it is not
cost-free:

* Process is considered every time the scheduler makes a decision
* Memory is occupied by the process, last load may have evicted other
  processes' memory from the cache

Massive concurrency slows down all processes.

---

# How can we not wait on I/O?

Blocking system calls cause this problem.

> Can we accomplish our desired tasks without blocking?

## Yes! Asynchronous I/O

* `select()`: Provided a list of file descriptors, block only until at least
  one is ready for I/O
* `epoll_*()`: Register to listen for events on file descriptors. Again block
  only until at least one of the registered descriptors is ready for I/O

---

# Select example

Assume we have a list of sockets called fd_list.

    loop forever:
        select(fd_list, ...) // block until something has I/O to handle
        for fd in fd_list
            if fd is ready for IO
                handle_io(fd)
            else do nothing

## some_handler

* can include socket acceptance
* shouldn't make any blocking calls
    * Use their non-blocking variants
    * Rely on select/epoll to tell you when I/O is ready
* should avoid excessive computation
    * Use a separate thread, process, or worker pool for such purposes

---

# Event Driven Systems

Systems that operate in such a manner are called event driven system.

Often such systems can accomplish everything using only a single process and
thread, of course more may be needed for CPU-bound segments.

Well used examples:

* NGINX
* Tengine (fork of NGINX)
* lighttpd
* netty (java)
* node.js (JavaScript)
* eventmachine (ruby)
* twisted (python)

---

# Event Driven HTTP Servers

## Strengths

* High performance under high load
* Predictable performance under high load
* No need to be thread-proof (unless specifically adding thread-concurrency)

# Weaknesses

* Poor isolation
    * What happens if a bug causes an infinite loop?
* Fewer extensions, since code cannot use blocking syscalls
* Very complex

---

# Event Machine (Ruby) Example

Event driven code is dominated by callbacks:

    !ruby
    EM.run {
      page = EM::HttpRequest.new('http://google.ca/').get
      page.errback { p "Google is down! terminate?" }
      page.callback {
        a = EM::HttpRequest.new('http://google.ca/search?q=em').get
        a.callback { # callback nesting, ad infinitum }
        a.errback  { # error-handling code }
      }
    }

---

# Callback Hell

![Yo dawg, I heard you like JavaScript](img/callback_hell.png)

It _can_ very easily become complicated.

---

# HTTP Server Architectures Review

* Single Threaded
* Process per request
    * Greatest isolation
    * Largest memory footprint
* Thread per request
    * Less isolation
    * Smaller memory footprint
* Process/thread worker pool
    * Tunable compromise between processes and threads
* Event-driven
    * Great performance under high load
    * Difficult to extend
    * Reduced isolation

---

# Application Servers

We are building web applications, so we will require complex server-side logic.

We _can_ extend our HTTP servers to provide this logic through modules, but
there are benefits to separating application servers to distinct process(es).

* Application logic will be dynamic
* Application logic regularly uses high level (slow) languages
* Security concerns are easier (HTTP server can shield app server from
  malformed requests)
* Setup costs can be amortized if the app server is running continuously

Instead we can have our HTTP server forward requests to the application
server(s).

---

# Inter-server Communication

> How does an HTTP Server communicate with the application server(s)?

A few different ways:

## CGI

Spawn a process, pass HTTP headers as ENV variables and utilize STDOUT as the
response.

## FastCGI, SCGI

Modifications to CGI to allow for persistent application server processes
(amortizes setup time).

## HTTP

Communicate via the HTTP protocol to a long-running process. (Essentially a
reverse-proxy configuration).

> Does it make sense to do this?

---

# Application Server Architectures

> What architecture should we use for our application server?

We have the same trade-offs to consider as with HTTP servers.

## Up Next

Let's take a quantitative look at various approaches with Ruby application
servers.

We will not consider evented ruby application servers (e.g., EventMachine)
because Rails will not run on such application servers.

---

# Our Test Setup

![Demo App](img/demo_app.png)

The Demo App is a link sharing website with:

* Multiple communities
* Each community can have many submissions
* Each submission can have a tree of comments

---

# Simulated Users

Using Tsung (erlang-based test framework) we will simulate multiple users
visiting the Demo App web service. Each user will:

    Visit the homepage (/)
      Wait randomly between 0 and 2 seconds
    Request community creation form
      Wait randomly between 0 and 2 seconds
    Submit new community form
    Request new link submission form
      Wait randomly between 0 and 2 seconds
    Submit new link submission form
      Wait randomly between 0 and 2 seconds
    Delete the link
      Wait randomly between 0 and 2 seconds
    Delete the community

---

# Test Process

There are six phases of testing each lasting 60 seconds:

1. (0-59s) Every second a new simulated user arrives
2. (60-119s) Every second 1.5 new simulated users arrive
3. (120-179s) Every second 2 new simulated users arrive
4. (180-239s) Every second 2.5 new simulated users arrive
5. (240-299s) Every second 3 new simulated users arrive
6. (300-359s) Every second 3.5 new simulated users arrive

__Note__: Each user corresponds to seven requests and a user may wait up to ten
seconds with the delays.

---

# Test Environment

All tests were conducted on a single Amazon EC2 m3-medium instance.

* 1 vCPU
* 3.75 GB RAM

The tests used the `Puma` web application server (unless otherwise specified).

The `database_optimizations` branch of the demo app was used to run the tests:
[https://github.com/scalableinternetservices/demo/tree/database_optimizations](https://github.com/scalableinternetservices/demo/tree/database_optimizations)

---

# Single Thread/Process (Users)

![Single Thread/Process Users](img/demo_single_users.png)

---

# Single Thread/Process (Page Load)

Decrease in performance around 60s (1.5 new users per second)

![Single Thread/Process Page Load](img/demo_single_page_load.png)

---

# Four Processes (Users)

![Four Processes Users](img/demo_four_users.png)

---

# Four Processes (Page Load)

Decrease in performance around 240s (3 new users per second)

![Four Processes Page Load](img/demo_four_load.png)

---

# Sixteen Processes (Users)

![Sixteen Processes Users](img/demo_sixteen_users.png)

---

# Sixteen Processes (Page Load)

Little improvement over 4 processes

![Sixteen Processes Page Load](img/demo_sixteen_load.png)

---

# Threads instead of processes?

> What do you think will happen?

---

# Four Threads (Users)

![Four Threads Users](img/demo_four_threads_users.png)

---

# Four Threads (Page Load)

Still decrease in performance around 240s, but more
stable until then.

![Four Threads Page Load](img/demo_four_threads_load.png)

---

# 32 Threads (Users)

![32 Threads Users](img/demo_32_threads_users.png)

---

# 32 Threads (Page Load)

Decrease in performance beginning around 300s (3.5 new users per second)

![32 Threads Page Load](img/demo_32_threads_load.png)

---

# Side note: Ruby interpreters

There are different versions of the Ruby interpreter. Different workloads may
benefit from using different interpreters.

## MRI (Matz's Ruby Interpreter)

* The reference version
* Written in C
* Has a global interpreter lock (GIL) that prevents true thread-concurrency

## JRuby

* Written in Ruby
* Does not have GIL

---

# Application Server Options

In this class you will be able to compare the performance of a handful of
application servers:

## via provided templates

* WEBrick (single process, single thread)
* Puma (worker pool)
* Phusion Passenger (worker pool)

## by creating your own templates

* Unicorn
* Thin
* Mongrel
* ???

---

# Puma

Originally designed for Rubinius (GIL-less ruby interpreter).

Claims to require less memory than others (the server itself)

Specifically made to work with thread-based parallelism, but also supports
multiple processes each with a tunable number of threads.



---

# Phusion Passenger

Passenger is a ruby web application server that can be added as a module to
either Apache or NGINX.

Passenger works as a worker pool adjusting the number of processes that
handle requests. Originally did not support threads within the processes.

---

# Puma and Passenger

Both Puma and Phusion Passenger:

* are relatively easy to configure
* enable processes to be forked after ruby/rails is loaded

> Why might you want to wait to load the application prior to forking?

---

# Thread Safety Note

If you can use thread-parallelism, do it! But, making your code thread safe
isn't always obvious.

## Old example: pp/models/order.rb

    !ruby
    class Order
      belongs_to :user, klass: User
    end


The above code may result in _autoloading_ the `user.rb` for the `User`
model. In versions of ruby prior to 2.0 (February 2013) autoloading was not
thread safe.

> What else might not be thread safe?

Things to consider:

* Your code
* Your many dependencies
