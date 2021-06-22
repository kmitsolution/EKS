# AutoScaler
<ul>
  <li>Responsible for dynamically scale the nodes in the node group</li>
  <li>Runs as deployment</li>
  <li>Decides on the basis of CPU & RAM availability</li>
  <li>mixture of on demand and spot instance possible</li>
</ul>  

### LAB
The cluster autoscaler automatically launches additional worker nodes if more resources are needed, and shutdown worker nodes if they are underutilized. The autoscaling works within a nodegroup, hence create a nodegroup first which has this feature enabled.

### Create AutoScale node group in AZ  by creating below yaml file (eks-auto.yaml)

        apiVersion: eksctl.io/v1alpha5
        kind: ClusterConfig

        metadata:
          name: EKS-raman-cluster
          region: us-east-1

        nodeGroups:
          - name: scale-east1c
            instanceType: t2.small
            desiredCapacity: 1
            maxSize: 5
            availabilityZones: ["us-east-1c"]
            iam:
              withAddonPolicies:
                autoScaler: true
            labels:
              nodegroup-type: stateful-east1c
              instance-type: onDemand
            ssh: # use existing EC2 key
              publicKeyName: vdevops
          - name: scale-east1d
            instanceType: t2.small
            desiredCapacity: 1
            maxSize: 5
            availabilityZones: ["us-east-1a"]
            iam:
              withAddonPolicies:
                autoScaler: true
            labels:
              nodegroup-type: stateful-east1d
              instance-type: onDemand
            ssh: # use existing EC2 key
              publicKeyName: vdevops
          - name: scale-spot
            desiredCapacity: 1
            maxSize: 5
            instancesDistribution:
              instanceTypes: ["t2.small", "t3.small"]
              onDemandBaseCapacity: 0
              onDemandPercentageAboveBaseCapacity: 0
            availabilityZones: ["us-east-1c", "us-east-1a"]
            iam:
              withAddonPolicies:
                autoScaler: true
            labels:
              nodegroup-type: stateless-workload
              instance-type: spot
            ssh:
              publicKeyName: vdevops

<b> Run Below command to create nodegroup with autoscaler enabled </b>

        eksctl create nodegroup --config-file=eks-auto.yaml
        # Run below command to check the nodes get created or not
        kubectl get nodes
        # Run below command to 
