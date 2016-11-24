## GlusterFS概览

### GlusterFS的适用性

GlusterFS非常适合不变的大型静态文件。在这种情况下“大”的意思是指兆字节（MB）及以上。示例：媒体文件，文档资产，图像。最好的情况是这些文件不会改变，例如，不像办公环境中共享存储器上的Office文档，会以同一个文件经常被打开和修改。

对于经常写小文件（千字节左右）甚至文件经常改变的Web服务器，GlusterFS不是一个很好的解决方案。不深究细节的情况下，问题通常是与竞争条件相关的，如其中两个（或更多个）GlusterFS服务器不同意特定文件的状态，所导致的所谓“裂脑（split brain）”场景。GlusterFS会采取一些步骤，尝试自动解决裂脑的问题，但没有完美的方式，总是能确定文件的哪个版本是正确的。

### GlusterFS的缺点

gluster文件系统上的底层文件只有通过外部挂载（客户端）才可见，不能ssh到gluster服务器上，直接在本地查看这些文件，在一些特殊场景，如GlusterFS运行非常缓慢时，想要查看和操作文件变得非常困难。

忠告：在“云应用”的世界中，我们应该首先考虑将数据从传统的文件系统中转移出来，使用缓存服务，对象存储，NoSQL解决方案和良好的旧数据库来保存状态和资产。参考：[Our GlusterFS Experience](http://www.catalyst.net.nz/blog/our-glusterfs-experiences)。
