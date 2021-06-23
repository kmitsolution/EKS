## Cloudwatch logging of an EKS cluster

<ul>
<li>Enable or Disable per log type</li>
<li>log types like api,audit,authenticator,control manager,scheduler</li>
<li>Each log type creates its own Cloud Watch log stream</li>
<li>name prefix /aws/eks/cluster-name</li>
<li>Logging adds some additional Cost to storage </li>
</ul>

### Create cloudwatch.yaml file for Cloud watch (We can enable/disable by using GUI)
```
apiVersion: eksctl.io/v1alpha5
kind: ClusterConfig

metadata:
  name: EKS-raman-cluster
  region: eu-north-1

nodeGroups:
  - name: ng-1
    instanceType: t3.small
    desiredCapacity: 3
    ssh: # use existing EC2 key
      publicKeyName: eunorth1
cloudWatch:
  clusterLogging:
    enableTypes: ["api", "audit", "authenticator"]
```

#### Enable cloudwatch logging
      eksctl utils update-cluster-logging --config-file cloudwatch.yaml --approve

#### disable via plain commandline call

```bash
eksctl utils update-cluster-logging --name=EKS-raman-cluster --disable-types all  --approve
```

