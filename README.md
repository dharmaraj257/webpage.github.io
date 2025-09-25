# **AWS DevOps Pipeline: GitHub → Jenkins → Ansible → Docker**

This project demonstrates an end-to-end CI/CD pipeline on AWS using Jenkins, Docker, and Ansible.
It automates the build, test, and deployment process, triggered by changes in the GitHub repository.
The pipeline ensures fast, reliable, and repeatable application delivery with minimal downtime.
<img width="1920" height="1080" alt="Auto-pulll" src="https://github.com/user-attachments/assets/d3a7b04c-84c0-4944-807f-8f5ad6cc07ec" />




### **Prerequisites:**

Before setting up this CI/CD pipeline, ensure you have:

**1. An AWS Account with at least 2 EC2 instances:**

  *  Jenkins Server – for CI/CD orchestration

  * Deployment Server – for running Docker containers via Ansible


**2.  Installed tools:**

* Jenkins (with Git, Pipeline, SSH Agent plugins)

* Docker & Docker Compose

* Ansible

* Git

**3. Networking & Access:**

* SSH key-based authentication between Jenkins and Deployment server

* Security group rules to allow: 22 (SSH), 80/443 (HTTP/HTTPS), 8080 (Jenkins)

# **Getting started:**

 **1. Launch AWS EC2 Instances**

* Create 2 Ubuntu servers on AWS (Jenkins + Docker).
<img width="1912" height="551" alt="image" src="https://github.com/user-attachments/assets/dc19355d-0c51-4bca-b45b-44b01fefe1ca" />



* Update security groups for required ports. <br>
<img width="1583" height="447" alt="image" src="https://github.com/user-attachments/assets/b54fa51c-7b14-4a93-801f-7de19bf13c2c" />


**2. Install Jenkins (on Jenkins Server)**
```

sudo apt update
sudo apt install -y openjdk-17-jdk git ansible
# Install Jenkins
curl -fsSL https://pkg.jenkins.io/debian-stable/jenkins.io-2023.key | sudo tee /usr/share/keyrings/jenkins-keyring.asc > /dev/null
echo deb [signed-by=/usr/share/keyrings/jenkins-keyring.asc] https://pkg.jenkins.io/debian-stable binary/ | sudo tee /etc/apt/sources.list.d/jenkins.list > /dev/null
sudo apt update
sudo apt install -y jenkins
sudo systemctl enable jenkins && sudo systemctl start jenkins

```
Access Jenkins: http://<Jenkins Server_PUBLIC_IP>:8080

<img width="940" height="503" alt="image" src="https://github.com/user-attachments/assets/8bba2093-9af0-4a6e-9b40-ae278cce13c4" />

**3. Install Ansible (on Jenkins Server)**
```
sudo apt update -y
sudo apt install software-properties-common -y
sudo add-apt-repository --yes --update ppa:ansible/ansible
sudo apt install ansible -y

```


**4. Install Docker (on Docker Server)**

```
sudo apt update
sudo apt install -y ca-certificates curl gnupg lsb-release
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg
echo "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.gpg] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
sudo apt update
sudo apt install -y docker-ce docker-ce-cli containerd.io docker-compose-plugin
sudo usermod -aG docker $USER
```
* Add current user in Docker Group.
  ```
  sudo usermod -aG docker ubuntu
  newgrp docker
  docker ps
  ```
* Install pip
  ```
  sudo apt install python3-pip
  ```
  
* Install Docker dependecies
  ```
  pip install docker
  ```
* Add server ip address in ansible host file.
* cd /etc/ansible
* sudo nano hosts
  ```
  [dockerserver]
  docker server ip address
  ```
  
**5. Setup SSH Access**

* ssh-keygen -t rsa -b 4096

* ssh-copy-id root@<DOCKER_SERVER_IP>

* paste in docker server authorized_keys 
* reolad ssh 
* Check access of server 
* ssh root@docker_server_ip

**6. Create a ansible playbook.(on jenkins servser)**
   sudo su →  su jenkins → mkdir playbooks → cd playbooks
   
   deployment.yaml
   ```
   - name: Build & Deploy Docker Container
    hosts: dockerservers
    gather_facts: false
    remote_user: root
    tasks:

    - name: Stop the old container if running
      docker_container:
        name: webpage.github.io-conatiner
        state: stopped
      ignore_errors: yes   # don’t fail if it doesn’t exist

    - name: Remove the old container
      docker_container:
        name: webpage.github.io-conatiner
        state: absent
      ignore_errors: yes

    - name: Remove old image (to avoid cache issues)
      docker_image:
        name: webpage.github.io
        tag: latest
        state: absent
        
    - name: Build a fresh Docker image
      docker_image:
        name: webpage.github.io
        tag: latest
        build:
          path: /root/project/
          pull: yes        
          nocache: yes     
        source: build
        state: present
        force_source: yes  

    - name: Run the new container
      docker_container:
        name: webpage.github.io-conatiner
        image: webpage.github.io:latest
        ports:
          - "80:80"
        state: started
        recreate: yes
   ```

**This playbook will:**

* Stop & remove old container

* Remove old Docker image (to avoid caching)

* Build a fresh Docker image from code copied by Jenkins

* Run a new container

**7.Setup Jenkins Pipeline (Freestyle Job)**

1. Create New Job
* Open Jenkins Dashboard → Click “New Item”
* Enter job name webpage.github.io
* Select Freestyle project → Click OK
2. Source Code Management
* Select Git
* Enter your repository URL.
* Specify branch to build.
3. Build Triggers
 * Poll SCM → Schedule:
   ```
   * * * * *
   ```
4. Environment
* Delete workspace before build starts
5. Build steps → Exceute shell
  Command 
   ```
   ssh -o StrictHostKeyChecking=no root@Server ip address "rm -rf ~/project/*"
   scp -r /var/lib/jenkins/workspace/webpage.github.io/* root@Server ip address:~/project 
    ansible-playbook /var/lib/jenkins/playbooks/deployment.yaml
   ```
6. Save & Build

* Click Save
* Run Build Now to test
* Check logs in Console Output

7. Verify Deployment
   * On Docker server
     ```
     docker ps
     ```
  * In Browser:
    ```
    http://Docker_SERVER_IP>
    ```
# webpage.github.io![webpage](https://user-images.githubusercontent.com/100831265/194479336-ef44b56c-626d-4bb5-bd0a-9098d225d1de.png)

**After this, every commit will:**
* Pull latest code from GitHub
* Clean Jenkins workspace
* Copy files to remote server
* Deploy via Ansible
  
# Features

* Automated build, test, and deployment pipeline
* GitHub integration
* Jenkins orchestrates CI/CD workflows
* Docker for containerized deployments
* Ansible for provisioning and deployment automation
* Runs on scalable AWS infrastructure

# Future Enhancements
* Add monitoring with Prometheus + Grafana
* Kubernetes integration for scaling

