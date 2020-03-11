---
layout: post
title:  "[6.824 Lab1] the filelist task assignment loop"
date:   2020-03-11 22:19:43 +0800
categories: go
---

today, I finished the job to start loop in worker to send rpc to master to get file task.


first, add IsOver to reply to store if the master's file task has all done, everytime master send a reply to worker, the filelist will -1

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
	fmt.Printf("#reply.filename: %v nRduce:%v\n", reply.Filename, reply.NReduce)
    
    for reply.IsOver == false {
    	call("Master.WorkerMsgHandler", &args, &reply)
    	fmt.Printf("reply.filename: %v nRduce:%v\n", reply.Filename, reply.NReduce)
    }

	fmt.Println("##### Finish this worker ########")
}
{% endhighlight %}

in master.go
{% highlight go linenos %}
func (m *Master) WorkerMsgHandler(args *MrArgs, reply *MrReply) error {
    m.mr.Lock()
    defer m.mr.Unlock()
    
    reply.NReduce = m.nReduce

    if m.files == nil {
    	reply.IsOver = true
    	reply.Filename = ""
    	return nil
    }

    if len(m.files) == 1 {
    	reply.Filename = m.files[0]
    	m.files = nil
    	reply.IsOver = true
    } else {
        reply.Filename, m.files = m.files[len(m.files)-1], m.files[:len(m.files)-1]
        reply.IsOver = false
    }
    return nil
}

func (m *Master) Done() bool {
	ret := false

	// Your code here.
	m.mr.Lock()
	defer m.mr.Unlock()

    if m.files == nil {
    	ret = true
    }

	return ret
}
{% endhighlight %}

When you use rpc, the master, as an RPC server, will be concurrent, so we need to lock the shared data in the handler(AKA, WorkerMsgHandler), so I add mr.lock() and unlock() in it.

but it does not fix the data race problem. I took the whole night, and finally I got the reason, you need to add lock everywhere you used the data which is shared, even in a function may not be called concurrently. (like in line28, in Done function)