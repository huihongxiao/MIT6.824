# 介绍
在这个系列的实验工作中，你最终会建立一个有容错机制的键值存储系统，这个实验是一系列实验的第一步。在这个实验中你会实现 Raft ，一个复制状态机协议。在下一个实验中你在 Raft 协议的基础上建立一个键值存储系统。之后你会在多个复制状态机上把你的存储服务进行“分库分表”，来取得更高的性能。

一个复制状态机通过在多个复制服务节点上存储全部的状态（比如数据）来达到容错的目的。这些备份允许服务在某些节点宕机的时候（网络故障，或者机器死机）仍然能正常工作。同时，这一性质带来了新的挑战，那就是不同节点可能存储了不一致的数据。

Raft 把客户端的请求组织成为一个序列，称为*日志*（log） ，并且保证所有的节点服务器看到的是相同的日志。每个节点按照日志顺序执行客户端的请求，把请求应用在状态在本节点的备份上。因为所有节点看到的是相同的客户端请求顺序，因此所有请求的执行顺序也相同，因此他们会拥有相同的状态。如果一个机器宕机恢复，Raft 会将其更新到最新的日志状态。只要多数节点还存活并且能相互沟通， Raft 就可以正常工作。如果多数节点都已经不能正常工作，Raft 将不会再有进一步的发展，但一旦多数节点能够再次沟通，Raft将恢复其中断的位置。

在这个 lab 中你会用 Go 语言实现 Raft 类以及它相关的方法，同时它会作为一个模块在更大的服务中被使用。一个 Raft 实例集群通过 RPC 彼此沟通，维护日志的复制同步。你的 Raft 借口将会支持一个无限期的包含被编号的指令的序列，同时也叫做*日志条目*。这些条目带有一个下标数字作为编号，一个带有下标的日志条目最终会被*提交*，在那时，你的 Raft 模块需要把这个日志条目送往更大的服务去执行。

你需要仿照[此论文](http://nil.csail.mit.edu/6.824/2020/papers/raft-extended.pdf)中的设计，尤其是要注意此论文中的*图 2*。你将会实现这个论文中的大部分内容，包括状态持久化和在节点重启恢复的时候从持久化设备中读取状态。你会实现集群成员变更（第六部分）。在稍后的实验中你会实现日志压缩/快照（第七部分）。

你或许会需要这个[博客](https://thesquareplanet.com/blog/students-guide-to-raft/)的帮助，同时你需要学习这些关于[锁](http://nil.csail.mit.edu/6.824/2020/labs/raft-locking.txt)和[结构](http://nil.csail.mit.edu/6.824/2020/labs/raft-structure.txt)的知识，来帮助你实现并发。如果你想进一步学习，可以去看 Paxos ， Chubby，Spanner，Zookkeeper，Harp，Viewstamped Replication，以及[这些内容](https://static.usenix.org/event/nsdi11/tech/full_papers/Bolosky.pdf)。

# 开始
raft 代码框架在`src/raft/raft.go`，相关的测试在`src/raft/test_test.go`。运行以下指令来开始，别忘了`git pull`一下来更新代码。
```bash
$ cd ~/6.824
$ git pull
...
$ cd src/raft
$ go test
Test (2A): initial election ...
--- FAIL: TestInitialElection2A (5.04s)
        config.go:326: expected one leader, got none
Test (2A): election after network failure ...
--- FAIL: TestReElection2A (5.03s)
        config.go:326: expected one leader, got none
...
$
```