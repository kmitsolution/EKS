# EKS Lab Setup (using Region Virginia (us-east-1))

<b> For Lab point of view we will setup the environment as per below diagram </b>

<img src="https://github.com/kmitsolution/EKS/blob/main/images/EKS_Setup1.PNG" width =700 height=400 />


<img src="https://github.com/kmitsolution/EKS/blob/main/images/EKS_Setup2.png" width =700 height=400 />

## Step 1:- Create a IAM user and EKS Role
   <ul>
    <li>Create the user with IAM with console and programmatic and download security credentials with security access id and Access key</li>
    <li>Permission point of view provide EKSFullAccess(There are some set of policies you need to add) and other permission as per requirement to deal EKS with other services  </li>
  <li> Create IAM Role for EKS <br />
    EKS (Allows EKS to manage clusters on your behalf.):- Role name :- AWSServiceRoleForAmazonEKS </br>
    EKS - Cluster  AWSClusterRoleForAmazonEKS</li>
      
  </ul>

## Step 2:- Create the ssh key pair
       Create SSH key value pair ( I am creating a key pair as vdevops)

## Step 3:- Setup Command line Tools

<ul>
  <li><b>Install awscli on Ubuntu Ec2 Instance</b> <br>
    apt install unzip -y <br/>
    curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip" <br />
    unzip awscliv2.zip<br />
    sudo ./aws/install<br />
    export PATH=/usr/local/bin:$PATH <br />
    aws --version<br />
    aws configure (To confiure aws credential with region=us-east-1)
  </li>
  <li>
    <b> Install and setup eksctl </b><br />
      curl --silent --location "https://github.com/weaveworks/eksctl/releases/download/latest_release/eksctl_$(uname -s)_amd64.tar.gz" | tar xz -C /tmp  <br />
      sudo mv /tmp/eksctl /usr/local/bin </br>
      eksctl version
  </li>    
  <li>
  <b> Install and setup kubectl </b><br />
     curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl" </br>
     sudo install -o root -g root -m 0755 kubectl /usr/local/bin/kubectl </br>
     kubectl version --client </br>

  </li>
</u>  
 
 ## Step 3:- Create first EKS Cluster using eksctl
 
 Create a yaml file to create EKS Cluster ( I have given file name as eks.yaml)
 
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
 
 ## Step 4:- Run below commands w.r.t eksctl
 
      # Find the available options for eksctl create cluster command
      eksctl create cluster --help
      # create cluster by using yaml config file (it will take 15-20 mins to create cluster )
      eksctl create cluster -f eks.yaml
      # Post cluster check
       kubectl get nodes
      # check there must be a /root/.kube/config file which refers to EKS Cluster
      
      # get cluster 
          eksctl get cluster
      # get node group details of the cluster
          eksctl get nodegroup --cluster EKS-raman-cluster
          
      

