## 存储卷
     1.volume 是pod 中能被多个容器访问的共享目录，生命周期和pod相同，
 
### emptyDir
     一个emptyDir vplume 是在pod分配node时创建的。它的初始内容为空，并且无须指定宿主机上对应的目录文件，因为这是kubernetes 自动分配的一个目录，当Pod 从Node 上移除时，emptyDir 中的数据也会被永久删除。      
     用途：
       临时空间，例如用于某些应用程序运行时所需的临时目录，且无须永久保留
       长时间任务的中间过程CheckPoint 的临时保存目录
       一个容器需要从另一个容器中获取数据的目录(多容器共享目录)

### hostPath 
     hostPath 为在Pod上挂载宿主机上的文件或目录，它通常可以用于以下几个方面。 
            