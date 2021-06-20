# Node Group Opertaions

### get node group details of the cluster
      eksctl get nodegroup --cluster EKS-raman-cluster

### scale out node group to 5 nodes and then check no of instances created in EC2
     eksctl scale nodegroup --cluster EKS-raman-cluster  --name ng-1 --nodes 5 --nodes-max 5
### scale in node group to 3 nodes and then check no of instances created in EC2
     eksctl scale nodegroup --cluster EKS-raman-cluster  --name ng-1 --nodes 3
### create a new node group with spot instance and demand instance 
<b> First create a yaml file as mentioned below (nodegroup.yml)
  
      apiVersion: eksctl.io/v1alpha5
      kind: ClusterConfig

      metadata:
        name: EKS-raman-cluster
        region: us-east-1

      nodeGroups:
        - name: ng-1
          instanceType: t2.small
          desiredCapacity: 3
          ssh: # use existing EC2 key
            publicKeyName: vdevops
        - name: ng-mixed
          minSize: 3
          maxSize: 5
          instancesDistribution:
            maxPrice: 0.2
            instanceTypes: ["t2.small", "t3.small"]
            onDemandBaseCapacity: 0
            onDemandPercentageAboveBaseCapacity: 50
          ssh:
            publicKeyName: vdevops
  
  ### Run below command to create ng-mixed node group and after that check in the ec2 instances.
  
        eksctl create nodegroup --config-file=nodegroup.yml --include='ng-mixed'
  
  
  ### Run below command to delete ng-mixed node group 
  
         eksctl delete nodegroup --config-file=nodegroup.yml --include=ng-mixed --approve


     
