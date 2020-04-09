---
layout: post
title:  "[go tips] how to use timer"
date:   2020-04-09 now
categories: go
---

### start a timer

```go
electionTimer *time.Timer

func (rf *Raft) setElectionTimer(){
    rand.Seed(time.Now().UnixNano())
    n := rand.Intn(150)+150  //a number between 150~300
    rf.electionDuration = time.Duration(n) * time.Millisecond
    rf.electionTimer = time.NewTimer(rf.electionDuration)
}
```

### stop a timer

```go
func (rf *Raft) stopElectionTimer(){
    //true means stop just now; false means already stopped or fired
    if !rf.electionTimer.Stop() {   
        <-rf.electionTimer.C  //drain the channel to ensure the channal is empty
    }
}
```

### reset a timer

```go
func (rf *Raft) resetElectionTimer(){
    if !rf.electionTimer.Stop() {
        //in 6.824 2A, we do not drain the channel here
        // we will discuss this at next
        //<-rf.electionTimer.C   
    }
    rf.electionTimer.Reset(rf.electionDuration)
}
```

### why we do not drain the channel at resetElectionTimer:

```go
func (rf *Raft) resetElectionTimer(){
    rf.mu.Lock()
    if !rf.electionTimer.Stop() {
        // if we drain .C here, then the follower will locked at line1, then the next election will never happen
        //<-rf.electionTimer.C  
    }
    rf.electionTimer.Reset(rf.electionDuration)
    rf.mu.Unlock()
}

//at the make function, we start a loop like this
go func() {
    for {
        rf.mu.Lock()
        state := rf.state
        term := rf.currentTerm
        me := rf.me
        rf.mu.Unlock()
        switch state {
        case FOLLOWER:
            rf.resetElectionTimer()   //line1
            select{
            case <- rf.heartBeatCh:
                DPrintf("follower rf %v heartbeat for other!\n", me)
                
            case <- rf.voteCh:
                DPrintf("follower rf %v voted for other!\n", me)
                
            case <- rf.electionTimer.C:
                DPrintf("follower rf %v timeout change to candidate\n", me)
                rf.changeState(CANDIDATE, term)
            }
        case CANDIDATE:
            DPrintf("[START ELECTION] rf %v start to election ....\n", me)
            rf.startElection()
            rf.resetElectionTimer()
            select {
            case <- rf.heartBeatCh:
                DPrintf("candidate rf %v heartbeat for other!\n", me)
                rf.changeState(FOLLOWER, term)
                
            case <- rf.voteCh:
                DPrintf("candidate rf %v voted for other!\n", me)
                rf.changeState(FOLLOWER, term)
                
            case <- rf.leaderCh:
                DPrintf("candidate rf %v is selected as leader!\n", me)
                
            case <- rf.electionTimer.C:
                DPrintf("candidate rf %v timeout change to candidate\n", me)
                rf.changeState(CANDIDATE, term)
            }
        case LEADER:
            DPrintf("[START HEARTBEAT] leader rf %v start to heartbeat ....\n", me)
            rf.stopElectionTimer()
            rf.startHeartBeat()
        }
        
    }
}()
```