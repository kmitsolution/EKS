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
        # Run below command to delete ng-1 node group
        eksctl delete nodegroup -f eks.yaml --include="ng-1"
        eksctl delete nodegroup -f eks.yaml --include="ng-1" --approve

<b> Deploy the AutoScaler </b>
    
        kubectl apply -f https://raw.githubusercontent.com/kubernetes/autoscaler/master/cluster-autoscaler/cloudprovider/aws/examples/cluster-autoscaler-autodiscover.yaml

<b> put required annotation to the deployment: </b>
        
        kubectl -n kube-system annotate deployment.apps/cluster-autoscaler cluster-autoscaler.kubernetes.io/safe-to-evict="false"

<b> get the autoscaler image version:  </b>

          open https://github.com/kubernetes/autoscaler/releases and get the latest release version matching your Kubernetes version, e.g. Kubernetes 1.14 => check for 1.14.n  where "n" is the latest release version. change cluster name (EKS-raman-cluster) and version of EKS
          
          kubectl -n kube-system edit deployment.apps/cluster-autoscaler


# Create nginx-deployment.yaml

        apiVersion: apps/v1

        kind: Deployment

        metadata:

          name: test-autoscaler

        spec:

          selector:

            matchLabels:

              app: nginx

          replicas: 1

          template:

            metadata:

              labels:

                service: nginx

                app: nginx

            spec:

              containers:

              - image: nginx

                name: test-autoscaler

                resources:

                  limits:

                    cpu: 300m

                    memory: 512Mi

                  requests:

                    cpu: 300m

                    memory: 512Mi

              nodeSelector:

                instance-type: spot

<b> Check the Log details </b>

      kubectl -n kube-system logs deployment.apps/cluster-autoscaler

  

