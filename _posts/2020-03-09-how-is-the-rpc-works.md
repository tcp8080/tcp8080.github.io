---
layout: post
title:  "[6.824 Lab1] how is the rpc works"
date:   2020-03-09 22:19:43 +0800
categories: go
---

At the first day reading the source code of 6.824 lab1:

{% highlight go linenos %}
//this is in master.go
func (m *Master) server() {
	rpc.Register(m)
	rpc.HandleHTTP()
	//l, e := net.Listen("tcp", ":1234")
	sockname := masterSock()
	os.Remove(sockname)
	l, e := net.Listen("unix", sockname)
	if e != nil {
		log.Fatal("listen error:", e)
	}
	go http.Serve(l, nil)  //start the server
}


// this is in worker.go
// send an RPC request to the master, wait for the response.
// usually returns true.
// returns false if something goes wrong.
//
func call(rpcname string, args interface{}, reply interface{}) bool {
	// c, err := rpc.DialHTTP("tcp", "127.0.0.1"+":1234")
	sockname := masterSock()
	c, err := rpc.DialHTTP("unix", sockname)  //start the client
	if err != nil {
		log.Fatal("dialing:", err)
	}
	defer c.Close()

	err = c.Call(rpcname, args, reply)  //client send msg to server, and get reply
	if err == nil {
		return true
	}

	fmt.Println(err)
	return false
}

{% endhighlight %}

at line11, go http.Serve(l, nil) means start http.Serve concurrently, which means when this server() function is invoked, the net.Listen function will start concurrently, waiting for the workers to send.

at line24 one client is started, and at line30 client will call rpcname(Master.XXXX) which is a function of server master)

By creating one master and several workers, we can exactly get the model in paper [MapReduce(2004)](https://pdos.csail.mit.edu/6.824/papers/mapreduce.pdf)

for rpc usage, read this [Go Rpc](https://juejin.im/post/5c4d664af265da61290a86d4)
