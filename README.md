# Terraform을 통한 Resource 생성 

### 작업 환경 
#### 설치 항목   
- Git
- Terraform
- Kubectl
- AWS-IAM-AUTHENICATOR
- Helm
- AWS Account setup

##### [준비작업 ]
1. Install Terraform   
Hashicorp 공식 사이트 참조 (https://learn.hashicorp.com/tutorials/terraform/install-cli)   
Install yum-config-manager to manage you repositories.   
```shell
[ec2-user@ip-10-100-0-124 ~]$ sudo yum install -y yum-utils
```
Use yum-config-manager to add the official HashiCorp Linux repository.   
```shell
[ec2-user@ip-10-100-0-124 ~]$ sudo yum-config-manager --add-repo https://rpm.releases.hashicorp.com/AmazonLinux/hashicorp.repo
Updating Subscription Management repositories.
Unable to read consumer identity

This system is not registered to Red Hat Subscription Management. You can use subscription-manager to register.

Adding repo from: https://rpm.releases.hashicorp.com/AmazonLinux/hashicorp.repo
```
Install   
```shell
[ec2-user@ip-10-100-0-124 ~]$ sudo yum install terraform
```
Verify the installation
```shell
[ec2-user@ip-10-155-101-209 ~]$ terraform -help
Usage: terraform [global options] <subcommand> [args]
```
2. Install Helm
Setup Guide : https://helm.sh/docs/intro/install
```shell
[ec2-user@ip-10-155-101-209 ~]$ curl -fsSL -o get_helm.sh https://raw.githubusercontent.com/helm/helm/main/scripts/get-helm-3
[ec2-user@ip-10-155-101-209 ~]$ chmod 700 get_helm.sh
[ec2-user@ip-10-155-101-209 ~]$ ./get_helm.sh
Downloading https://get.helm.sh/helm-v3.7.2-linux-amd64.tar.gz
Verifying checksum... Done.
Preparing to install helm into /usr/local/bin
helm installed into /usr/local/bin/helm
[ec2-user@ip-10-155-101-209 ~]$
```
Helm 2.x와 달리 Helm 3.x에서는 기본 repository가 설정되어 있지 않다. 따라서 아래와 같이 repo를 추가한다.   
```shell
[ec2-user@ip-10-155-101-209 ~]$ helm repo add stable https://charts.helm.sh/stable
"stable" has been added to your repositories
[ec2-user@ip-10-155-101-209 ~]$ helm repo update
Hang tight while we grab the latest from your chart repositories...
...Successfully got an update from the "stable" chart repository
Update Complete. ⎈Happy Helming!⎈
```
3. Install Kubectl
```shell
[ec2-user@ip-10-155-101-209 ~]$ curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl"
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
100   154  100   154    0     0    608      0 --:--:-- --:--:-- --:--:--   608
100 44.4M  100 44.4M    0     0  25.6M      0  0:00:01  0:00:01 --:--:-- 46.4M
[ec2-user@ip-10-155-101-209 ~]$ sudo install -o root -g root -m 0755 kubectl /usr/local/bin/kubectl
[ec2-user@ip-10-155-101-209 ~]$ kubectl version --client
Client Version: version.Info{Major:"1", Minor:"23", GitVersion:"v1.23.0", GitCommit:"ab69524f795c42094a6630298ff53f3c3ebab7f4", GitTreeState:"clean", BuildDate:"2021-12-07T18:16:20Z", GoVersion:"go1.17.3", Compiler:"gc", Platform:"linux/amd64"}
```
4. Install AWS-IAM-AUTHENICATOR
Kubectl로 AWS EKS에 접근하기 위해서는 AWS-IAM-AUTHENICATOR가 필요하다.   
Installation URL : https://docs.aws.amazon.com/ko_kr/eks/latest/userguide/install-aws-iam-authenticator.html
```shell
[ec2-user@ip-10-155-101-209 ~]$ curl -o aws-iam-authenticator https://amazon-eks.s3-us-west-2.amazonaws.com/1.21.2/2021-07-05/bin/linux/amd64/aws-iam-authenticator
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
100 33.6M  100 33.6M    0     0  3859k      0  0:00:08  0:00:08 --:--:-- 4713k
[ec2-user@ip-10-155-101-209 ~]$ chmod +x ./aws-iam-authenticator
[ec2-user@ip-10-155-101-209 ~]$ mkdir -p $HOME/bin && cp ./aws-iam-authenticator $HOME/bin/aws-iam-authenticator && export PATH=$PATH:$HOME/bin
[ec2-user@ip-10-155-101-209 ~]$ echo 'export PATH=$PATH:$HOME/bin' >> ~/.bashrc
[ec2-user@ip-10-155-101-209 ~]$ aws-iam-authenticator help
A tool to authenticate to Kubernetes using AWS IAM credentials

Usage:
  aws-iam-authenticator [command] 
```
5. AWS 계정
AWS 계정이 필요하고 terraform-eks 관련된 권한이 필요하다. 
```shell
[ec2-user@ip-10-155-101-209 ~]$ aws configure
AWS Access Key ID [None]: xxxxxxxx
AWS Secret Access Key [None]: xxxxxxxx
Default region name [None]: ap-northeast-2
Default output format [None]:
[ec2-user@ip-10-155-101-209 ~]$
```
Terraform이 AWS EKS에 접근이 가능하도록 IAM 퍼미션을 추가한다. 
```json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Sid": "VisualEditor0",
            "Effect": "Allow",
            "Action": [
                "autoscaling:AttachInstances",
                "autoscaling:CreateAutoScalingGroup",
                "autoscaling:CreateLaunchConfiguration",
                "autoscaling:CreateOrUpdateTags",
                "autoscaling:DeleteAutoScalingGroup",
                "autoscaling:DeleteLaunchConfiguration",
                "autoscaling:DeleteTags",
                "autoscaling:Describe*",
                "autoscaling:DetachInstances",
                "autoscaling:SetDesiredCapacity",
                "autoscaling:UpdateAutoScalingGroup",
                "autoscaling:SuspendProcesses",
                "ec2:AllocateAddress",
                "ec2:AssignPrivateIpAddresses",
                "ec2:Associate*",
                "ec2:AttachInternetGateway",
                "ec2:AttachNetworkInterface",
                "ec2:AuthorizeSecurityGroupEgress",
                "ec2:AuthorizeSecurityGroupIngress",
                "ec2:CreateDefaultSubnet",
                "ec2:CreateDhcpOptions",
                "ec2:CreateEgressOnlyInternetGateway",
                "ec2:CreateInternetGateway",
                "ec2:CreateNatGateway",
                "ec2:CreateNetworkInterface",
                "ec2:CreateRoute",
                "ec2:CreateRouteTable",
                "ec2:CreateSecurityGroup",
                "ec2:CreateSubnet",
                "ec2:CreateTags",
                "ec2:CreateVolume",
                "ec2:CreateVpc",
                "ec2:CreateVpcEndpoint",
                "ec2:DeleteDhcpOptions",
                "ec2:DeleteEgressOnlyInternetGateway",
                "ec2:DeleteInternetGateway",
                "ec2:DeleteNatGateway",
                "ec2:DeleteNetworkInterface",
                "ec2:DeleteRoute",
                "ec2:DeleteRouteTable",
                "ec2:DeleteSecurityGroup",
                "ec2:DeleteSubnet",
                "ec2:DeleteTags",
                "ec2:DeleteVolume",
                "ec2:DeleteVpc",
                "ec2:DeleteVpnGateway",
                "ec2:Describe*",
                "ec2:DetachInternetGateway",
                "ec2:DetachNetworkInterface",
                "ec2:DetachVolume",
                "ec2:Disassociate*",
                "ec2:ModifySubnetAttribute",
                "ec2:ModifyVpcAttribute",
                "ec2:ModifyVpcEndpoint",
                "ec2:ReleaseAddress",
                "ec2:RevokeSecurityGroupEgress",
                "ec2:RevokeSecurityGroupIngress",
                "ec2:UpdateSecurityGroupRuleDescriptionsEgress",
                "ec2:UpdateSecurityGroupRuleDescriptionsIngress",
                "ec2:CreateLaunchTemplate",
                "ec2:CreateLaunchTemplateVersion",
                "ec2:DeleteLaunchTemplate",
                "ec2:DeleteLaunchTemplateVersions",
                "ec2:DescribeLaunchTemplates",
                "ec2:DescribeLaunchTemplateVersions",
                "ec2:GetLaunchTemplateData",
                "ec2:ModifyLaunchTemplate",
                "ec2:RunInstances",
                "eks:CreateCluster",
                "eks:DeleteCluster",
                "eks:DescribeCluster",
                "eks:ListClusters",
                "eks:UpdateClusterConfig",
                "eks:UpdateClusterVersion",
                "eks:DescribeUpdate",
                "eks:TagResource",
                "eks:UntagResource",
                "eks:ListTagsForResource",
                "eks:CreateFargateProfile",
                "eks:DeleteFargateProfile",
                "eks:DescribeFargateProfile",
                "eks:ListFargateProfiles",
                "eks:CreateNodegroup",
                "eks:DeleteNodegroup",
                "eks:DescribeNodegroup",
                "eks:ListNodegroups",
                "eks:UpdateNodegroupConfig",
                "eks:UpdateNodegroupVersion",
                "iam:AddRoleToInstanceProfile",
                "iam:AttachRolePolicy",
                "iam:CreateInstanceProfile",
                "iam:CreateOpenIDConnectProvider",
                "iam:CreateServiceLinkedRole",
                "iam:CreatePolicy",
                "iam:CreatePolicyVersion",
                "iam:CreateRole",
                "iam:DeleteInstanceProfile",
                "iam:DeleteOpenIDConnectProvider",
                "iam:DeletePolicy",
                "iam:DeletePolicyVersion",
                "iam:DeleteRole",
                "iam:DeleteRolePolicy",
                "iam:DeleteServiceLinkedRole",
                "iam:DetachRolePolicy",
                "iam:GetInstanceProfile",
                "iam:GetOpenIDConnectProvider",
                "iam:GetPolicy",
                "iam:GetPolicyVersion",
                "iam:GetRole",
                "iam:GetRolePolicy",
                "iam:List*",
                "iam:PassRole",
                "iam:PutRolePolicy",
                "iam:RemoveRoleFromInstanceProfile",
                "iam:TagOpenIDConnectProvider",
                "iam:TagRole",
                "iam:UntagRole",
                "iam:TagPolicy",
                "iam:TagInstanceProfile",
                "iam:UpdateAssumeRolePolicy",
                // Following permissions are needed if cluster_enabled_log_types is enabled
                "logs:CreateLogGroup",
                "logs:DescribeLogGroups",
                "logs:DeleteLogGroup",
                "logs:ListTagsLogGroup",
                "logs:PutRetentionPolicy",
                // Following permissions for working with secrets_encryption example
                "kms:CreateAlias",
                "kms:CreateGrant",
                "kms:CreateKey",
                "kms:DeleteAlias",
                "kms:DescribeKey",
                "kms:GetKeyPolicy",
                "kms:GetKeyRotationStatus",
                "kms:ListAliases",
                "kms:ListResourceTags",
                "kms:ScheduleKeyDeletion"
            ],
            "Resource": "*"
        }
    ]
}
```
위의 정책을 하나 생성하고 AWS 사용자 계정에 연결한다.   
그리고 ELB의 정보를 확인하기 위해서 아래 내용의 IAM 정책을 하나 더 만들어 AWS 사용자 계정에 연결한다.   
```json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Sid": "VisualEditor0",
            "Effect": "Allow",
            "Action": [
                "elasticloadbalancing:DescribeLoadBalancers"
            ],
            "Resource": "*"
        }
    ]
}
```
6. Git 설치    
```shell
[ec2-user@ip-10-155-101-209 ~]$ sudo yum update -y
[ec2-user@ip-10-155-101-209 ~]$ sudo yum install git -y
[ec2-user@ip-10-155-101-209 ~]$ git version
git version 2.32.0
```
### EKS 생성 
#### Sample Terraform 스크립트 준비
```shell
[ec2-user@ip-10-155-101-209 ~]$ git clone https://github.com/terraform-providers/terraform-provider-aws.git
Cloning into 'terraform-provider-aws'...
remote: Enumerating objects: 311550, done.
remote: Counting objects: 100% (1040/1040), done.
remote: Compressing objects: 100% (471/471), done.
remote: Total 311550 (delta 661), reused 870 (delta 562), pack-reused 310510
Receiving objects: 100% (311550/311550), 316.34 MiB | 21.68 MiB/s, done.
Resolving deltas: 100% (219089/219089), done.
[ec2-user@ip-10-155-101-209 ~]$ cd terraform-provider-aws/examples/eks-getting-started/
```
디렉토리의 파일들을 확인해보면 .tf 확장자 파일들이 존재하는 것을 확인할 수 있다. 이 파일들이 terraform 스크립트이며 샘플 EKS 클러스터를 구성하도록 작성되어 있다.   
```shell
[ec2-user@ip-10-155-101-209 eks-getting-started]$ ls -al
total 36
drwxrwxr-x  2 ec2-user ec2-user  178 Dec 16 04:02 .
drwxrwxr-x 27 ec2-user ec2-user 4096 Dec 16 04:02 ..
-rw-rw-r--  1 ec2-user ec2-user 2047 Dec 16 04:02 eks-cluster.tf
-rw-rw-r--  1 ec2-user ec2-user 1580 Dec 16 04:02 eks-worker-nodes.tf
-rw-rw-r--  1 ec2-user ec2-user 1032 Dec 16 04:02 outputs.tf
-rw-rw-r--  1 ec2-user ec2-user  397 Dec 16 04:02 providers.tf
-rw-rw-r--  1 ec2-user ec2-user  519 Dec 16 04:02 README.md
-rw-rw-r--  1 ec2-user ec2-user  131 Dec 16 04:02 variables.tf
-rw-rw-r--  1 ec2-user ec2-user 1160 Dec 16 04:02 vpc.tf
-rw-rw-r--  1 ec2-user ec2-user  476 Dec 16 04:02 workstation-external-ip.tf
[ec2-user@ip-10-155-101-209 eks-getting-started]$
```
variables.tf 파일을 살펴보면 aws_region의 default값이 있는데 해당 region으로 변경한다. 또한 cluster-name도 변경한다.
```shell
$ cat variables.tf
variable "aws_region" {
  default = "ap-northeast-2"
}

variable "cluster-name" {
  default = "eks-kaltour-cluster"
  type    = string
}
```
##### EKS 생성   
1. Terraform Initialization
Terraform을 사용하기 위해 초기화를 하는 작업이다. 
```shell
[ec2-user@ip-10-155-101-209 eks-getting-started]$ terraform init

Initializing the backend...

Initializing provider plugins...
- Finding latest version of hashicorp/aws...
- Finding latest version of hashicorp/http...
- Installing hashicorp/aws v3.69.0...
- Installed hashicorp/aws v3.69.0 (signed by HashiCorp)
- Installing hashicorp/http v2.1.0...
- Installed hashicorp/http v2.1.0 (signed by HashiCorp)

Terraform has created a lock file .terraform.lock.hcl to record the provider
selections it made above. Include this file in your version control repository
so that Terraform can guarantee to make the same selections by default when
you run "terraform init" in the future.

Terraform has been successfully initialized!

You may now begin working with Terraform. Try running "terraform plan" to see
any changes that are required for your infrastructure. All Terraform commands
should now work.

If you ever set or change modules or backend configuration for Terraform,
rerun this command to reinitialize your working directory. If you forget, other
commands will detect it and remind you to do so if necessary.
```
2. Terraform Plan
.tf 파일의 내용을 실제로 적용 가능한지 확인하는 작업이다.
```shell
[ec2-user@ip-10-155-101-209 eks-getting-started]$ terraform plan

Terraform used the selected providers to generate the following execution plan. Resource actions are indicated with the following
symbols:
  + create

Terraform will perform the following actions:
```
3. Terraform Apply
.tf 파일의 내용대로 리소스를 생성, 수정, 삭제하는 작업을 한다.
```shell
[ec2-user@ip-10-155-101-209 eks-getting-started]$ terraform apply
```

#### Kubectl 설정 
EKS 생성이 완료되면 kubectl을 이용해서 EKS에 접근해야 한다. 아래 명령어를 통해 접근할 EKS의 정보를 yaml 형태의 kubectl을 yaml config로 출력할 수 있다.
```shell
[ec2-user@ip-10-155-101-209 eks-getting-started]$ terraform output kubeconfig
<<EOT


apiVersion: v1
clusters:
- cluster:
    server: https://DC287DC1511002BD5427B7E7B53365AD.gr7.ap-northeast-2.eks.amazonaws.com
    certificate-authority-data: LS0tLS1CRUdJTiBDRVJUSUZJQ0FURS0tLS0tCk1JSUM1ekNDQWMrZ0F3SUJBZ0lCQURBTkJna3Foa2lHOXcwQkFRc0ZBREFWTVJNd0VRWURWUVFERXdwcmRXSmwKY201bGRHVnpNQjRYRFRJeE1USXhOakExTVRBeU5Wb1hEVE14TVRJeE5EQTFNVEF5TlZvd0ZURVRNQkVHQTFVRQpBeE1LYTNWaVpYSnVaWFJsY3pDQ0FTSXdEUVlKS29aSWh2Y05BUUVCQlFBRGdnRVBBRENDQVFvQ2dnRUJBTE9TCk5OeXNrNStTS1B6NHIxTHVjaU0zUnNmaHNJcTJFVzc5VjhjY1NFRGJmdjZLNTErcVR1WENNcThMVFZlMkw5RGUKb3NZRHdHM1MwSWM1R3lvZGZaNXVCNSttN3IyZzhYU0FTYURDVHhQTTZDQXVhaWJ5TUpkV3ZKWVB0bmtvQmhUdgpIcU05NitaM21HNGcyeGFQbzJ4VUJtSWNVVXhYYk95Ulg5L200bXF2WkhmL0piQ29vL0ZkckV4aDFVZ2NmOTRzCjZqa2JKclZDb084L0pQdGtvU3J3NFkvMkhHMm1iOExZRXdFbCs1eS91THI3eTAxT0l4bmNjVzJoZHdzZzdzbDkKUWpMREplM1V0UFd3YjZobE1kSFM0UFVCM2YvL0ZrRXhlWno3WEN6cWwxUG1sdlVTclJkK3d5elVFdWJ4TEpWMwpPMXd3RVUzdWNnYnVvaHRqeTFFQ0F3RUFBYU5DTUVBd0RnWURWUjBQQVFIL0JBUURBZ0trTUE4R0ExVWRFd0VCCi93UUZNQU1CQWY4d0hRWURWUjBPQkJZRUZLam1vSkc4QkdWd0kwWkc1Um56Y01ObVNLbW9NQTBHQ1NxR1NJYjMKRFFFQkN3VUFBNElCQVFBYkFtV1dQdGdXdmxQT1RlcVhzMnczcThkNkMzYkttNlBRblVMVFRtdTVkYUEzOVdwVApsT1FDc1hXaWx1Tno3eVBiMGxhY2hJUXQzVXJtcGM5aDZRWmxpS1RVSjI5WkpiQXZBM21kYmxSaFJOUGh1T1lCCkFzZThUbzVndTNKWEFlTVFJNmVueFR6T0hqRDFKRmdiQWhDV3MvNzNLZUxjNWk2YmRLNjhSRTRDZ2NhcGJuUXcKU0FqM3ZSQkd3SUdGZGUyaCtPQnl1akwrMGpXb2NlQkwrSVViTzBuSkc5VTJKRkRUbnV3SlMyUjl5OTdYaUozWgpHWWRVS2tHL1VQM2NoTklBTFJHRzBaZkdjUUpDd2J6TGJlcFp4Uy9HMFNmbFJaZG5VM01xZCtZYVphNlRvSFdVCi9yOE8xa0preFk2SnZWMkRlNG1xWS9RbFp3Q2psbEhPQWVWMwotLS0tLUVORCBDRVJUSUZJQ0FURS0tLS0tCg==
  name: kubernetes
contexts:
- context:
    cluster: kubernetes
    user: aws
  name: aws
current-context: aws
kind: Config
preferences: {}
users:
- name: aws
  user:
    exec:
      apiVersion: client.authentication.k8s.io/v1alpha1
      command: aws-iam-authenticator
      args:
        - "token"
        - "-i"
        - "eks-kaltour-cluster"

EOT
[ec2-user@ip-10-155-101-209 eks-getting-started]$
```
그 정보를 kubectl의 설정파일에 입력하면 된다.    
config 파일에 EOF 내용을 제거해야 된다.  
```shell
[ec2-user@ip-10-155-101-209 eks-getting-started]$ mkdir ~/.kube/
[ec2-user@ip-10-155-101-209 eks-getting-started]$ terraform output kubeconfig > ~/.kube/config
```
완료가 되면 확인 작업을 위해 kubectl 버전을 확인해서 Client, Server의 버전 정보가 잘 출력되었는지 확인한다.
```shell
[ec2-user@ip-10-155-101-209 eks-getting-started]$ kubectl version
Client Version: version.Info{Major:"1", Minor:"23", GitVersion:"v1.23.0", GitCommit:"ab69524f795c42094a6630298ff53f3c3ebab7f4", GitTreeState:"clean", BuildDate:"2021-12-07T18:16:20Z", GoVersion:"go1.17.3", Compiler:"gc", Platform:"linux/amd64"}
Server Version: version.Info{Major:"1", Minor:"21+", GitVersion:"v1.21.2-eks-06eac09", GitCommit:"5f6d83fe4cb7febb5f4f4e39b3b2b64ebbbe3e97", GitTreeState:"clean", BuildDate:"2021-09-13T14:20:15Z", GoVersion:"go1.16.5", Compiler:"gc", Platform:"linux/amd64"}
WARNING: version difference between client (1.23) and server (1.21) exceeds the supported minor version skew of +/-1
[ec2-user@ip-10-155-101-209 eks-getting-started]$
```
#### ConfigMap 접근 설정
Kubernetes Master 노드의 ConfigMap에 대해 EKS 클러스터가 접근할 수 있도록 설정을 한다.    
EOF 문자를 제거해야 한다.
```shell
[ec2-user@ip-10-155-101-209 eks-getting-started]$ terraform output config_map_aws_auth > configmap.yaml
[ec2-user@ip-10-155-101-209 eks-getting-started]$ vi configmap.yaml
[ec2-user@ip-10-155-101-209 eks-getting-started]$ kubectl apply -f configmap.yaml
Warning: resource configmaps/aws-auth is missing the kubectl.kubernetes.io/last-applied-configuration annotation which is required by kubectl apply. kubectl apply should only be used on resources created declaratively by either kubectl create --save-config or kubectl apply. The missing annotation will be patched automatically.
configmap/aws-auth configured
[ec2-user@ip-10-155-101-209 eks-getting-started]$
```
#### EKS 재배포 
##### Worker Node 개수 변경
현재 worker node의 개수를 확인한다. 
```shell
[ec2-user@ip-10-155-101-209 eks-getting-started]$ kubectl get nodes -o wide
NAME                                            STATUS   ROLES    AGE   VERSION               INTERNAL-IP   EXTERNAL-IP   OS-IMAGE         KERNEL-VERSION                CONTAINER-RUNTIME
ip-10-0-0-117.ap-northeast-2.compute.internal   Ready    <none>   94m   v1.21.5-eks-bc4871b   10.0.0.117    3.38.96.21    Amazon Linux 2   5.4.156-83.273.amzn2.x86_64   docker://20.10.7
ip-10-0-0-41.ap-northeast-2.compute.internal    Ready    <none>   94m   v1.21.5-eks-bc4871b   10.0.0.41     3.36.59.11    Amazon Linux 2   5.4.156-83.273.amzn2.x86_64   docker://20.10.7
ip-10-0-1-126.ap-northeast-2.compute.internal   Ready    <none>   94m   v1.21.5-eks-bc4871b   10.0.1.126    3.37.44.243   Amazon Linux 2   5.4.156-83.273.amzn2.x86_64   docker://20.10.7
```
eks-worker-nodes.tf 파일을 확인하면 desired_size, max_size, min_size가 3으로 설정된 것을 볼 수 있다. 
```shell
resource "aws_eks_node_group" "demo" {
  cluster_name    = aws_eks_cluster.demo.name
  node_group_name = "demo"
  node_role_arn   = aws_iam_role.demo-node.arn
  subnet_ids      = aws_subnet.demo[*].id

  scaling_config {
    desired_size = 3
    max_size     = 5
    min_size     = 3
  }

  depends_on = [
    aws_iam_role_policy_attachment.demo-node-AmazonEKSWorkerNodePolicy,
    aws_iam_role_policy_attachment.demo-node-AmazonEKS_CNI_Policy,
    aws_iam_role_policy_attachment.demo-node-AmazonEC2ContainerRegistryReadOnly,
  ]
}
```
원하는 숫자로 수정하고 다시 재배포 해본다.
```shell
$ terraform plan
$ terraform apply
```
#### Helm을 이용한 POD 배포 
Helm 2.x를 사용중이면 Helm Tiller를 Kubernetes에 ServiceAccount로 배포해야 한다.   
하지만 Helm 3.x를 사용하면 더 이상 Tiller 설치가 필요하지 않다.    
##### nginx-ingress 배포 
Helm을 이용해서 ingress를 배포한다. 
```shell
[ec2-user@ip-10-155-101-209 eks-getting-started]$ helm install my-nginx stable/nginx-ingress --set rbac.create=true
WARNING: Kubernetes configuration file is group-readable. This is insecure. Location: /home/ec2-user/.kube/config
WARNING: Kubernetes configuration file is world-readable. This is insecure. Location: /home/ec2-user/.kube/config
WARNING: This chart is deprecated
NAME: my-nginx
LAST DEPLOYED: Thu Dec 16 06:55:02 2021
NAMESPACE: default
STATUS: deployed
REVISION: 1
TEST SUITE: None
NOTES:
```
해당 nginx-ingress는 deprecated 되었지만 Terraform, Helm을 이용한 EKS 구성을 테스트하기는 적합한다.   
아래와 같이 ingress 관련 2개의 pod가 running 중인 것을 확인할 수 있다. 
```shell
[ec2-user@ip-10-155-101-209 eks-getting-started]$ kubectl get pod
NAME                                                      READY   STATUS    RESTARTS   AGE
my-nginx-nginx-ingress-controller-85bdc54c58-drtkh        1/1     Running   0          89s
my-nginx-nginx-ingress-default-backend-7c95885dd5-bs7lk   1/1     Running   0          89s
```

##### Application 배포 
airflow라는 어플리케이션을 배포해서 테스트해본다.   
아래와 같이 airflow 관련 manifest.yaml 파일을 생성한다.   
ingress가 / 경로 호출에 대해 airflow로 라우팅하도록 설정하고 있다. 
```shell
$ cat > airflow-manifest.yaml <<EOF
ingress:
  enabled: true
  web:
    path: "/"
    annotations:
      kubernetes.io/ingress.class: "nginx"
EOF
```
helm으로 배포한다.   
```shell
[ec2-user@ip-10-155-101-209 eks-getting-started]$ helm install airflow stable/airflow -f airflow-manifest.yaml
```
아래와 같이 airflow 관련 6개의 pod가 추가된 것을 확인할 수 있다. 
```shell
[ec2-user@ip-10-155-101-209 eks-getting-started]$ kubectl get pod
NAME                                                      READY   STATUS    RESTARTS   AGE
airflow-flower-857c757c46-lcjjs                           1/1     Running   0          63s
airflow-postgresql-0                                      1/1     Running   0          63s
airflow-redis-master-0                                    1/1     Running   0          63s
airflow-scheduler-776988c58c-zv5hw                        1/1     Running   2          63s
airflow-web-5cc644f669-qbhgk                              1/1     Running   2          63s
airflow-worker-0                                          1/1     Running   0          63s
my-nginx-nginx-ingress-controller-85bdc54c58-drtkh        1/1     Running   0          5m40s
my-nginx-nginx-ingress-default-backend-7c95885dd5-bs7lk   1/1     Running   0          5m40s
```
##### 서비스 확인
위에서 AWS 계정에서 ELB의 정보를 조회할 수 있는 권한설정을 다루었다.    
설정되었다면 아래 명령어로 ELB의 DNS-NAME을 확인할 수 있다.    
```shell
[ec2-user@ip-10-155-101-209 eks-getting-started]$ aws elb describe-load-balancers | grep DNSName
            "DNSName": "a812d2c1c88cb41a196cf6db8a93235e-1512615423.ap-northeast-2.elb.amazonaws.com",
[ec2-user@ip-10-155-101-209 eks-getting-started]$
```
##### Resources 제거  
###### Pod 제거 
아래 helm 명령어로 pod들을 제거 가능하다.
````shell
$ helm uninstall airflow
$ helm uninstall my-nginx
````
##### EKS 제거 
EKS와 같은 terraform 으로 생성한 리소스들은 아래 명령어로 제거 가능한다. 
```shell
$ terraform destroy
```
