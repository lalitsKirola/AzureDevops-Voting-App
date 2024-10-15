
	1. Create Azure devops project Voting App in an organisation
	
	2. Import repository from github voting app
	3. Create Vote service pipeline 
We can use template of pipeline for various processes like in this case we have used build and push image to container registry template .
	
	# Docker
	# Build and push an image to Azure Container Registry
	# https://docs.microsoft.com/azure/devops/pipelines/languages/docker
	trigger:
	 paths:
	   include:
	     - vote/*
	resources:
	- repo: self
	variables:
	  # Container registry service connection established during pipeline creation
	  dockerRegistryServiceConnection: '28cb4dde-cb8c-48b6-a3b7-5d5eb9d54972'
	  imageRepository: 'votingapp'
	  containerRegistry: 'lalitazurecicd.azurecr.io'
	  dockerfilePath: '$(Build.SourcesDirectory)/vote/Dockerfile'
	  tag: '$(Build.BuildId)'
	  # Agent VM image name
	pool:
	 name: 'azureagent'
	stages:
	- stage: Build
	  displayName: Build
	  jobs:
	  - job: Build
	    displayName: Build
	    steps:
	    - task: Docker@2
	      displayName: Build
	      inputs:
	        containerRegistry: '$(dockerRegistryServiceConnection)'
	        repository: '$(imageRepository)'
	        command: 'build'
	        Dockerfile: 'vote/Dockerfile'
	        tags: '$(tag)'
	- stage: Push
	  displayName: Push
	  jobs:
	  - job: Push
	    displayName: Push to ACR
	    steps:
	    - task: Docker@2
	      displayName: Push
	      inputs:
	        containerRegistry: '$(dockerRegistryServiceConnection)'
	        repository: '$(imageRepository)'
	        command: 'push'
	        tags: '$(tag)'
	- stage: Update
	  displayName: Update
	  jobs:
	  - job: Update
	    displayName: Update
	    steps:
	    # Step 1: Convert script line endings to Unix format using dos2unix
	    - script: |
	        dos2unix scripts/updateK8sManifests.sh
	      displayName: 'Convert script line endings to Unix format'
	    # Step 2: Run the shell script
	    - task: ShellScript@2
	      inputs:
	        scriptPath: 'scripts/updateK8sManifests.sh'
	        args: 'vote $(imageRepository) $(tag)'
	      displayName: "Run updateK8sManifests.sh"
	
	
	
	3. Then result service pipeline 
	
	# Docker
	# Build and push an image to Azure Container Registry
	# https://docs.microsoft.com/azure/devops/pipelines/languages/docker
	trigger:
	 paths:
	  include:
	    - result/*
	resources:
	- repo: self
	variables:
	  # Container registry service connection established during pipeline creation
	  dockerRegistryServiceConnection: '88b48a0f-d48f-4080-9597-315920d9e10b'
	  imageRepository: 'resultapp'
	  containerRegistry: 'lalitazurecicd.azurecr.io'
	  dockerfilePath: '$(Build.SourcesDirectory)/result/Dockerfile'
	  tag: '$(Build.BuildId)'
	  # Agent VM image name
	pool:
	 name: 'azureagent'
	stages:
	- stage: Build
	  displayName: Build
	  jobs:
	  - job: Build
	    displayName: Build
	    steps:
	    - task: Docker@2
	      displayName: Build
	      inputs:
	        containerRegistry: '$(dockerRegistryServiceConnection)'
	        repository: '$(imageRepository)'
	        command: 'build'
	        Dockerfile: 'result/Dockerfile'
	        tags: '$(tag)'
	- stage: Push
	  displayName: push stage
	  jobs:
	  - job: Push
	    displayName: Push
	    steps:
	    - task: Docker@2
	      displayName: Push
	      inputs:
	        containerRegistry: '$(dockerRegistryServiceConnection)'
	        repository: '$(imageRepository)'
	        command: 'push'
	        tags: '$(tag)'
	- stage: Update
	  displayName: Update
	  jobs:
	  - job: Update
	    displayName: Update
	    steps:
	    # Step 1: Convert script line endings to Unix format using dos2unix
	    - script: |
	        dos2unix scripts/updateK8sManifests.sh
	      displayName: 'Convert script line endings to Unix format'
	    # Step 2: Run the shell script
	    - task: ShellScript@2
	      inputs:
	        scriptPath: 'scripts/updateK8sManifests.sh'
	        args: 'result $(imageRepository) $(tag)'
	      displayName: "Run updateK8sManifests.sh"
	
	
	4. Worker service pipeline:
	
	# Docker
	# Build and push an image to Azure Container Registry
	# https://docs.microsoft.com/azure/devops/pipelines/languages/docker
	trigger:
	 paths:
	   include:
	     - worker/*
	resources:
	- repo: self
	variables:
	  # Container registry service connection established during pipeline creation
	  dockerRegistryServiceConnection: '3e0fc8dc-a937-4d87-aa4d-67885139ecd6'
	  imageRepository: 'workerapp'
	  containerRegistry: 'lalitazurecicd.azurecr.io'
	  dockerfilePath: '$(Build.SourcesDirectory)/worker/Dockerfile'
	  tag: '$(Build.BuildId)'
	  # Agent VM image name
	pool:
	 name: 'azureagent'
	stages:
	- stage: Build
	  displayName: Build
	  jobs:
	  - job: Build
	    displayName: Build
	    steps:
	    - task: Docker@2
	      displayName: Build
	      inputs:
	        containerRegistry: '$(dockerRegistryServiceConnection)'
	        repository: '$(imageRepository)'
	        command: 'build'
	        Dockerfile: 'worker/Dockerfile'
	        tags: '$(tag)'
	- stage: Push
	  displayName: Push
	  jobs:
	  - job: Push
	    displayName: Push
	    steps:
	    - task: Docker@2
	      displayName: Push
	      inputs:
	        containerRegistry: '$(dockerRegistryServiceConnection)'
	        repository: '$(imageRepository)'
	        command: 'push'
	        tags: '$(tag)'
	- stage: Update
	  displayName: Update
	  jobs:
	  - job: Update
	    displayName: Update
	    steps:
	    # Step 1: Convert script line endings to Unix format using dos2unix
	    - script: |
	        dos2unix scripts/updateK8sManifests.sh
	      displayName: 'Convert script line endings to Unix format'
	    # Step 2: Run the shell script
	    - task: ShellScript@2
	      inputs:
	        scriptPath: 'scripts/updateK8sManifests.sh'
	        args: 'worker $(imageRepository) $(tag)'
	      displayName: "Run updateK8sManifests.sh"
	        
	
	5. Create a VM for running azure pipeline build (agent pool in azure devops terminology), preferably ubuntu image
	Install docker.io after update it 
	Then adding azureuser to docker group.
	bash
	Copy code
	sudo usermod -aG docker azureuser
	This command:
		• Uses sudo to run as a superuser.
		• usermod is used to modify the user account.
		• The -aG flag appends (-a) the user to the group (-G) without removing them from other groups.
		• docker is the group you are adding the user to.
		• azureuser is the user being added to the group.
	Why add azureuser to the Docker group?
		• By adding azureuser to the docker group, you allow that user to run Docker commands without needing to use sudo each time.
		• Normally, Docker requires superuser privileges to run, so this is a common practice to streamline workflows.

	6. Connecting agent to azure devops pipeline
	
	
	
	
	
	
	Don’t just copy the command ~/$ you will get error , just do it one by one like creating myagent directory and cd into it and 
	then doing wget download link '
	https://vstsagentpackage.azureedge.net/agent/3.245.0/vsts-agent-linux-x64-3.245.0.tar.gz '
	
	Then

	tar zxvf ~/Downloads/vsts-agent-linux-x64-3.245.0.tar.gz
	
	Then
	./config.sh
	
	Then
	./run.sh to make the agent online or run the agent . 
	
	
	We can use git bash also to connect to a VM , just download the ssh key and then in git bash
	
	 ssh -i azureagent_key1.pem azureuser@20.235.243.196  (external ip of vm)
	
	It will ask for PAT (Personal Access Tokens ) create one token in Azure Devops with required acesss full or custom.
	
	
	You can see build being executed with steps.
	
	
	 
	
	
	
	Type logout in git bash you will exit the vm and vm will become offline.
	
	
	
	Commands I executed in VM:
	Last login: Mon Oct 14 02:29:17 2024 from 103.248.94.197
	azureuser@azureagent:~$ history
	    1  sudo su
	    2  mkdir myagent && cd myagent
	    3  tar zxvf ~/Downloads/vsts-agent-linux-x64-3.245.0.tar.gz
	    4  wget tar zxvf ~/Downloads/vsts-agent-linux-x64-3.245.0.tar.gz
	    5  ls
	    6  cd myagent/
	    7  ls
	    8  wget https://vstsagentpackage.azureedge.net/agent/3.245.0/vsts-agent-linux-x64-3.245.0.tar.gz
	    9  ls
	   10  tar zxvf ~/Downloads/vsts-agent-linux-x64-3.245.0.tar.gz
	   11  tar zxvf vsts-agent-linux-x64-3.245.0.tar.gz
	   12  sudo apt update
	   13  ls
	   14  ./config.sh
	   15  ls
	   16  ./run.sh
	   17  sudo install docker.io
	   18  sudo apt install docker.io
	   19  sudo usermod -aG docker azureuser
	   20  systemctl restart docker
	   21  sudo systemctl restart docker
	   22  ./run.sh
	   23  ls
	   24  ./config.sh
	   25  ./ru
	   26  ./run.sh
	   27  docker status
	   28  sudo systemctl status docker
	   29  sudo systemctl restart docker
	   30  ./run.sh
	   31  history
	   32  sudo usermod -aG docker azureuser
	   33  sudo systemctl restart docker
	   34  logout
	   35  cd myagent/
	   36  ls
	   37  ./run.sh
	   38  logout
	   39  cd myagent/
	   40  ls
	   41  ./run.sh
	   42  cd myagent/
	   43  ls
	   44  ./run.sh
	   45  sudo apt update
	   46  ./run.sh
	   47  sudo apt-get install dos2unix
	   48  ./run.sh
	   49  dos2unix scripts/updateK8sManifests.sh
	   50  ./run.sh
	   51  logout
	   52  cd myagent/
	   53  ls
	   54  ./run.sh
	   55  ls
	   56  cd bin
	   57  ls
	   58  ls -ltr
	   59  cd docker
	   60  ccd
	   61  cd
	   62  cd /usr/bin/docker
	   63  cd /usr
	   64  cd bin
	   65  cd docker
	   66  cd
	   67  cd myagent/
	   68  ./run.sh
	   69  ./run.sh
	   70  cd myagent/
	   71  ./run.sh
	   72  cd myagent/
	   73  ll
	   74  cd ./r
	   75  ./run.sh
	   76  logout
	   77  history
	azureuser@azureagent:~$
	



7      Create Azure container Registry.

When we execute the 3 pipeline all these repostory will be shown here  with there tag id.










	8. Create AKS Cluster (Don’t forget to enable public ip as we need to expose it through nodeport for using argocd)
To login to aks cluster:
	az aks get-credentials --name azuredevops --overwrite-existing --resource-group azurecicd
	
	
	
	
	----Install Argocd 
	kubectl create namespace argocd 
	kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml
	
	--- get argocd svc and change the cluster IP service with edit command to NodePort for Argocd-server service.
	
	
	
	
	
	Link for Argocd: externalip of node :nodeport of argocd-server service
	https://52.143.96.89:31349/
	
	
	
	username will be admin and for password edit argocd secret (argocd-initial-admin-secret) and copy the base64 code from there (like this MnNZc0hqcWpmaXk0QUl1OA==) .
	
	Then decode it with .
	  echo 'based64 code' | base64 --decode 
	You will get a decode value like this.
2sYsHjqjfiy4AIu8
	
	
	
	9. Create update script to edit new tag id in deployement yaml after pushing that updated image to ACR for a service.
	
	
	---------------------------------------------------
	#!/bin/bash
	set -x
	# Set the repository URL
	REPO_URL="https://rb6dpkscc4clw4qiddrvj7rhptbior6onmp2j4hvkvwxl6m3qufq@dev.azure.com/learnmeghwithlalit/voting-app/_git/voting-app"
	# Clone the git repository into the /tmp directory
	git clone "$REPO_URL" /tmp/temp_repo
	# Navigate into the cloned repository directory
	cd /tmp/temp_repo
	# Make changes to the Kubernetes manifest file(s)
	# For example, let's say you want to change the image tag in a deployment.yaml file
	sed -i "s|image:.*|image: lalitazurecicd.azurecr.io/$2:$3|g" k8s-specifications/$1-deployment.yaml
	# Add the modified files
	git add .
	# Commit the changes
	git commit -m "Update Kubernetes manifest"
	# Push the changes back to the repository
	git push
	# Cleanup: remove the temporary directory
	rm -rf /tmp/temp_repo
	
	
	---------------------------------------------------
	There will be issues related to  '\r' command not found issue during updation of the manifest files in the Update stage. This issue exists for windows users and can be fixed by running the below shell commands. sudo apt-get install dos2unix dos2unix <script.sh> // script file name And then use the generated script in repo.
	
	
	Logs for this script in azure devops pipeline
	Starting: Run updateK8sManifests.sh 
	============================================================================== 
	Task : Shell script 
	Description : Run a shell script using Bash 
	Version : 2.245.1 
	Author : Microsoft Corporation 
	Help : https://docs.microsoft.com/azure/devops/pipelines/tasks/utility/shell-script 
	============================================================================== 
	/usr/bin/bash /home/azureuser/myagent/_work/2/s/scripts/updateK8sManifests.sh vote votingapp 42 
	+ REPO_URL= https://rb6dpkscc4clw4qiddrvj7rhptbior6onmp2j4hvkvwxl6m3qufq@dev.azure.com/learnmeghwithlalit/voting-app/_git/voting-app 
	+ git clone https://rb6dpkscc4clw4qiddrvj7rhptbior6onmp2j4hvkvwxl6m3qufq@dev.azure.com/learnmeghwithlalit/voting-app/_git/voting-app /tmp/temp_repo 
	Cloning into '/tmp/temp_repo'... 
	+ cd /tmp/temp_repo 
	+ sed -i 's|image:.*|image: lalitazurecicd.azurecr.io/votingapp:42|g' k8s-specifications/vote-deployment.yaml 
	+ git add . 
	+ git commit -m 'Update Kubernetes manifest' 
	[main 1ad1066] Update Kubernetes manifest 
	Committer: Ubuntu <azureuser@azureagent.xqhnncv1jwou5n5mjwzaqrek3h.rx.internal.cloudapp.net> 
	Your name and email address were configured automatically based 
	on your username and hostname. Please check that they are accurate. 
	You can suppress this message by setting them explicitly. Run the 
	following command and follow the instructions in your editor to edit 
	your configuration file: 
	
	git config --global --edit 
	
	After doing this, you may fix the identity used for this commit with: 
	
	git commit --amend --reset-author 
	
	1 file changed, 1 insertion(+), 1 deletion(-) 
	+ git push 
	To https://dev.azure.com/learnmeghwithlalit/voting-app/_git/voting-app 
	84ebece..1ad1066 main -> main 
	+ rm -rf /tmp/temp_repo 
	
	Finishing: Run updateK8sManifests.sh 
	
	
	As soon as this scripts updates image tag from ACR to these deployement yamls , Argocd will redeploy the yamls as it continuously looks for any updates in the yamls .
	
	
	
	
	9. Configure argocd setup after logging into it .
Configure repostory it will look for any changes and path of the directory.
Then create a application like Voting app.

	
	Underlined token is PAT for connecting azure devops to argocd .
	
	
	
	
	
	
	
	
	
	
	
	
	
	
	
	
	
	
	
	
	
	
	
	
	10. 
	
	Command to create ACR ImagePullSecret
	kubectl create secret docker-registry <secret-name> \
    --namespace <namespace> \
    --docker-server=<container-registry-name>.azurecr.io \
    --docker-username=<service-principal-ID> \
    --docker-password=<service-principal-password>
	
	
	kubectl create secret docker-registry acr-secret     --namespace default     --docker-server=lalitazurecicd.azurecr.io     --docker-username=lalitazurecicd     --docker-password=WxLFG9caZGelU9ouSMU8Ug6sMzojF+BgJd/74/EwJi+ACRALt6I0
	
	Password is secret key that we  generated in ACR.
	
	
	
	
	Update the deployemnt yamls also refering the secret otherwise it will give imagepullback error.
	
	
	  
![image](https://github.com/user-attachments/assets/74f7919c-eb94-48cc-a0e1-6dd81e224b2e)
