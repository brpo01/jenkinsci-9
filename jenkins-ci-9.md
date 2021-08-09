# **BUILDING A CONTINUOUS INTEGRATION PIPELINE WITH JENKINS**

This project will cover the practicalities of building a jenkins pipeline that would run the build, test and deploy phases of a website solution.

What is Jenkins?. Jenkins is an open source automated server for automating all sorts of tasks such as building, testing & deploying software. Devops as a field is about automation and speedy releases of software while maintaining code quality. Jenkins can help us achieve this.  

There also has to be an understanding about what continuous integration is about. Continuous Integration is the automated testing and building of your app on every commit, this is made possible by configuring a bulid trigger(webhooks) to build the code on every commit, this eliminates the manual task of having to run the builds after a new commit is made. Also, what is continuous deployment. It is the automation of the build, test & deploy phases. If all the tests pass, the new commit will be pushed from the development environment to the production environment without manual intervention.

The reference architecture is shown below.

![1](https://user-images.githubusercontent.com/47898882/128691239-b8c28e84-9497-4776-8023-d9006553db7b.JPG)

# **Spin up an EC2 Instance & Install Jenkins & all of it's dependencies**

- Create an AWS EC2 server based on Ubuntu Server 20.04 LTS.

- Install Java Devlopment Kit on your server as Jenkins is a java based application.

```
$ sudo apt update
$ sudo apt install default-jdk-headless
```
- Install Jenkins & its dependecies

```
$ wget -q -O - https://pkg.jenkins.io/debian-stable/jenkins.io.key | sudo apt-key add -
$ sudo sh -c 'echo deb https://pkg.jenkins.io/debian-stable binary/ > /etc/apt/sources.list.d/jenkins.list'
$ sudo apt update
$ sudo apt-get install jenkins
```
- Restart the jenkins server and make sure it is running

```
$ sudo systemctl restart jenkins
$ sudo systemcel status jenkins
```

- Open your browser and paste in your public ipddress and the port 8080 in order to load the jenkins server

*Note: Do not forget to to allow inbound connections into port 8080, this is the port the jenkins server runs on*

- When the page is loaded, there'll be a prompt for an admin password. Recover the password from the `/var/lib/jenkins/secrets/initialAdminPassword` directory.

![2](https://user-images.githubusercontent.com/47898882/128691242-46888a48-a6e4-46b6-aa55-c1db5cf8b9b1.JPG)

- Install all suggested plugins as requested by jenkins

![3](https://user-images.githubusercontent.com/47898882/128691323-62424846-d9c0-4558-a317-7585a81a694b.JPG)

![4](https://user-images.githubusercontent.com/47898882/128691231-128df385-66a7-4711-96d1-d766aae3e49d.JPG)


# **Configure Jenkins to retrieve source codes from GitHub using Webhooks**

- Go to the [tooling](https://github.com/darey-io/tooling) repository and fork the code to your account. When that is done go to the settings of that repository and create a webhook in this format `publicip-address:port>/github-webhook`.

![6](https://user-images.githubusercontent.com/47898882/128693317-390ecfda-7715-4860-904c-54180a2692fb.JPG)

- Create a new freestyle project, and name it whatever you want.

![5](https://user-images.githubusercontent.com/47898882/128691234-c45dea85-13a0-4943-8996-e4c2eb85f0d8.JPG)

- A page to configure your freestyle project ill be loaded, add the url of the github repository and run the build.

![8](https://user-images.githubusercontent.com/47898882/128694095-78b62380-34d4-40bf-95da-e663d021d7f4.JPG)

![9](https://user-images.githubusercontent.com/47898882/128694100-90fa8830-214e-4be4-88a0-9fe832893dc8.JPG)

- The build should run successfully if everything was configured properly.

![10](https://user-images.githubusercontent.com/47898882/128694452-17768bfd-5013-45ab-be02-af72ebdce7b2.JPG)

- Now, there has to be a way to automate this process in the sense that if a new commit is made, a build is triggered and the build is run. How can this be achieved? well by using the github webhook that we created. Go to the build trigger section of your freestyle project and choose the option to of the github hook trigger.

![8](https://user-images.githubusercontent.com/47898882/128694759-340337a5-7ba5-4bd6-813a-3e311ad866cc.JPG)

- Configure "Post-build Actions" to archive all the files – files resulted from a build are called "artifacts". ** in this context means copy all the files in the repository.

![11](https://user-images.githubusercontent.com/47898882/128695451-99b93d4f-49f7-410b-a2c9-6b2c031fb684.JPG)

- Make a change in the your repository and commit the changes, you'll find out that the build will be run in an automated manner without you having to click the build now button. The Build should run successfully and your artifacts should be archived if the project is configured properly.

![13](https://user-images.githubusercontent.com/47898882/128695829-c10fa2d3-9a52-47ee-970d-9ab4c525204c.JPG)


![12](https://user-images.githubusercontent.com/47898882/128695573-38a1f9b8-fc35-44de-8165-881f16ca5282.JPG)

- You can find the artifacts in the /var/lib/jenkins/jobs/tooling_github/builds/<build_number>/archive/. Replace `tooling_github` with the name of your freestyle project.

# **CONFIGURE JENKINS TO COPY FILES TO NFS SERVER VIA SSH**

- The artifacts are stored locally on the jenkins server, and we can copy these files/folders into the nfs server using ssh. If you know about ssh, you now that it is used to establish remote connectivity between servers in an encrypted manner. To achieve this, the `Publish Over SSH`
plugin has to be installed

![14](https://user-images.githubusercontent.com/47898882/128696788-8c43d8d5-5c2c-48a7-a934-be4007c9047c.JPG)

- When Installation is done, configure the job/project to copy artifacts over to NFS server. On main dashboard select "Manage Jenkins" and choose "Configure System" menu item. Scroll down to Publish over SSH plugin configuration section and configure it to be able to connect to your NFS server. 

1. Provide a private key (content of .pem file that you use to connect to NFS server via SSH/Putty)
2. Hostname – can be private IP address of your NFS server
3. Username – ec2-user (since NFS server is based on EC2 with RHEL 8)
4. Remote directory – /mnt/apps since our Web Servers use it as a mointing point to retrieve files from the NFS server

![16](https://user-images.githubusercontent.com/47898882/128697537-62fc0574-4947-44a4-aeef-8bcddfb12414.JPG)

- Save the configuration and add another Post-build Action. This time pick the *Send Build Artifacts Over SSH* option. Use ** to copy all of the files and folders to the /mnt/apps directory on the nfs server.

![17](https://user-images.githubusercontent.com/47898882/128697962-508dce18-e33c-4676-b11d-99852b49821b.JPG)

- Make a change in your repository and commit the changes. A build will be triggered automatically and the build will be run. The outcome of the build should be successful if all the configurations were done properly. You'll that all of the files have been copied over to the nfs server successfully in your console output. To confirm chec the /mnt/apps directory on your nfs server

![18](https://user-images.githubusercontent.com/47898882/128698333-5b02bb51-a4f5-4c06-85fa-d95edeb4a085.JPG)

### Congratulations!!. You have built a jenkins freestyle project and implemented continuous integration for your software solution.





