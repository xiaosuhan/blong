### calico 
     
     1.查看IP池 
      calicoctl get ippool -o yaml   
	      apiVersion: projectcalico.org/v3
			items:
			- apiVersion: projectcalico.org/v3
			  kind: IPPool
			  metadata:
			    creationTimestamp: 2018-09-26T11:06:11Z
			    name: default-ipv4-ippool
			    resourceVersion: "161060"
			    uid: 2d3408b1-c17c-11e8-851d-525469f66d5d
			  spec:
			    cidr: 192.168.0.0/16
			    ipipMode: Always
			    natOutgoing: true
			kind: IPPoolList
			metadata:
			  resourceVersion: "546759"
	     