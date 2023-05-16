# Automate the Deployment of a Web Application (Nodejs) on Amazon EKS using Terraform to have a fully automated CI/CD solution.
- Create customized image for jenkins with docker and kubectl installed on it.
- Use bastion host (jump host public ec2 ) to access private worker node .
 
##  Infrastructure is created on aws using terraform.
### To run terraform infrastructre 

```bash
terraform init
terraform apply
```


### From my machine to jump host maschine or using ansible

#### Copy key and script file
```bash
  chmod +x file.sh
  scp -i final-key.pem file.sh ubuntu@44.203.72.128:/home/ubuntu  => to install aws and kubectl on jump host
  scp -i final-key.pem final-key.pem ubuntu@44.203.72.128:/home/ubuntu => to be able to connect with private worker node
``` 
#### Copy deployment files for jenkins

```bash
  scp -i final-key.pem deployment.yaml ubuntu@44.203.72.128:/home/ubuntu
  scp -i final-key.pem service.yaml ubuntu@44.203.72.128:/home/ubuntu
  scp -i final-key.pem serviceAccount.yaml ubuntu@44.203.72.128:/home/ubuntu
  scp -i final-key.pem volume.yaml ubuntu@44.203.72.128:/home/ubuntu
```
#### Connent to jump host machine

```bash
  ssh -i "final-key.pem" ubuntu@44.203.72.128
```
  
### From jump host machine

#### Connent to worker node machine (private)
```bash
ssh -i "final-key.pem" ec2-user@10.0.3.30 
```

##### From worker node machine enable docker

```bash
 sudo yum install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin 
 sudo systemctl start docker
 sudo systemctl status docker   
 exit
```

#### Install aws and kubectl
 ```bash
  sh file.sh  
```
#### Connect with eks cluster:
  ```bash
  aws configure 
  aws_access_key_id = 
  aws_secret_access_key =
  aws eks --region us-east-1 update-kubeconfig --name eks-cluster
  kubectl get nodes => to check that you connected with cluster
  kubectl get svc 
  ```
  
  #### Deploy deployment files for jenkins in a namespace
 
  ```bash
  kubectl create namespace jenkins
  kubectl apply -f volume.yaml -n jenkins
  kubectl apply -f deployment.yaml -n jenkins
  kubectl apply -f service.yaml -n jenkins
  kubectl apply -f serviceAccount.yaml -n jenkins
  kubectl get all -n jenkins 
  kubectl logs jenkins-7b586bdbcd-9dl9q -n jenkins 
  kubectl exec -it jenkins-7b586bdbcd-9dl9q bash 
  cat /var/jenkins_home
  ```
  
  #### Open jenkins
  - Put cedentials for docker hub
  - Create pipeline
  
  #### Install ingress controller
 
  ```bash
  kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/controller-v1.7.1/deploy/static/provider/cloud/deploy.yaml
  kubectl get all -n ingress-nginx

  kubectl get ingress -n app 
  
```

### Configuration using Ansible

1- Create config file in ~/.ssh/config as worker node is private

``` bash 
Host jump-host
        hostname 44.202.101.185
        user ubuntu
        port 22
        identityfile /home/marwa/Desktop/Ansible/Task/final-key.pem    
```
2- Then , run ansible
``` bash 
   ansible-galaxy init roles/docker
   ansible-galaxy init roles/aws-cli
   ansible-galaxy init roles/kubectl
   ansible-playbook playbook.yaml -i inventory.txt    
        
```











