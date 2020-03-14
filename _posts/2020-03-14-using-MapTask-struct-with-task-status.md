---
layout: post
title:  "[6.824 Lab1] using MapTask struct with task status"
date:   2020-03-14 now
categories: go
---

Today, I finally finished my Maptask job.

3 things are important:
1. task status need to be stored, or else the task may be duplicated assignd.
2. workers to tell master that it has finished the map task master assignned to it.
3. when the master received the worker's telling done, set the task status to done

Here is the MapTask struct, the maptasks is a map of key:filename, value:maptask(with taskid and task status)

```go
type Master struct {
	// Your definitions here.
	mr sync.Mutex
	nReduce int
	maptasks map[string]*MapTask
	isAllMapTaskDone bool
}
type MapTask struct {
    taskId int32
    status int  // 0:init, 1:ongoing, 2:finish
}
```

When the master is created, all the maptasks' status is set to 0, means task need to be assign.

When master received the worker RPC assigntask request, master will find a unassigned task, set its status as ongoing, and then tell the worker the task's filename and taskid.

When woreker get the master's response with taskid and filename, it will call Domap to start the real Map process.

And anytime you start a worker, it will automatically connect to the master and ask for a task, this will make the whole job easy to explore, from a little worker to whatever you want, as long as you have the worker resource.

here is the task status setting process:

```go
func (m *Master) initMapTask(files [] string) {
    for _, file := range files {
    	maptask := MapTask{}
    	maptask.taskId = rand.New(rand.NewSource(time.Now().UnixNano())).Int31n(1000000)
    	maptask.status = 0 //init
    	m.maptasks[file] = &maptask
    }
}

func (m *Master) assignTask() (string, int32){  //retrun filename, taskId
    if m.isAllMapTaskDone {
    	return "", -1
    }

    for file, mtask := range m.maptasks {
        if mtask.status == 0 { //not assigned
        	mtask.status = 1
            return file, mtask.taskId
        }
    }

    return "", -1
}

func (m *Master) doneTask(taskid int32) {
	for filename, mtask := range m.maptasks {
		if taskid == mtask.taskId {
			m.maptasks[filename].status = 2
		}
	}
}
```

I have done a lot of work for the task assignment, at first, all the Maptask job finish took a whole 15 seconds, and I used both files and maptasks to do the work, it really is a mass, two struct to store the very related things are quit confusing. Also there is no status to check very quickly if the task is done, I need to do a circulate, all these extra things waste quite a lot of time.

Finally, with the simple map to store all the maptask infomation, this makes the task process quite simple and clear, aslo the cost time is only 4 seconds, that's almost 1/3 of the old time. so this is really a good way.

Tomorrow I will try to add the costing time into the struct, and when the worker's takes more than 10 seconds to finish the task, master will consider it has failed, and will reassign the task.