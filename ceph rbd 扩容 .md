### ceph rbd 扩容 
   
    rbd showmapped   
    df -h   查看/dev/rbdx   的大小 
    扩容  
    rbd  resize --size    1024   foo   
    rbd resize --size  1024  liuguanghuif.CID-516874818ed4.datadir-lk-test-1  -p tenx-pool 
    df -Th   查看大小是否变化
    如果文件系统还未增大
    blockdev --getsize64 /dev/rbd4 
    resize2fs /dev/rbd4 
    
    查看大小ok！ 
    
    注意:对于xfs 要在resize 之后执行   xfs_growfs /mnt
     
      
      
   