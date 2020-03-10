---
layout: post
title:  "[6.824 Lab1] create your own RPC defination"
date:   2020-03-09 22:19:43 +0800
categories: go
---

today, I try to start master.go and worker.go with the simple thing:
1. master.go is run with parameters files, which is slice of the to MapReduce files.
2. worker.go send args to master and get the last filename of master's files

by running serveral workers each time, the filenames will be printed not in the same order, which means the rpc between every worker and master is parallelly setted.

in rpc.go
{% highlight go linenos %}
// Add your RPC definitions here.
type MrArgs struct {
    Taskid int  //the Taskid must begin with uppercase T, not t
}

type MrReply struct {
    Filename string  
}
{% endhighlight %}

in master.go
{% highlight go linenos %}
// Your code here -- RPC handlers for the worker to call.
func (m *Master) WorkerMsgHandler(args *MrArgs, reply *MrReply) error {
    m.mt.Lock()
    reply.Filename, m.files = m.files[len(m.files)-1], m.files[:len(m.files)-1]
    defer m.mt.Unlock()
    return nil
}
{% endhighlight %}


in worker.go
{% highlight go linenos %}
//
// main/mrworker.go calls this function.
//
func Worker(mapf func(string, string) []KeyValue,
	reducef func(string, []string) string) {

	// Your worker implementation here.
	args := MrArgs{}
	reply := MrReply{}

	call("Master.WorkerMsgHandler", &args, &reply)

	fmt.Printf("reply.filename: %v\n", reply.Filename)
}
{% endhighlight %}

### Q and A
#### Q: gob: type MyArgs has no exported fields rpc.call
A: make sure the parameter's name is begin with upper letter

#### Q:WARNING: DATA RACE
A: it should work with mutext's lock and unlock, but it does not work like what I did in master.go. I will dig it out why.