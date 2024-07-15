
## Accessing Git Repo: 
Access the Git Repo by using your browser and pointing it to: 
```
http://infra.5g-deployment.lab:3000/
```

You will see the initial Git Screen as a result: 

Use the "SignIn" Option, and the credentials `student` , `student` to login to Git: 

![images](images/git_3.png)

## Creating an empty ZTP repository: 

Using the `New Repository` option (as shown in figure below), start the creation of a new empty repository: 

![images](images/git_4.png)

Name the repository `ztp-repository` , as shown in following figure: 

![images](images/git_5.png)

Upon successful creation of repository, you will see a message similiar to the following: 

![images](images/git_6.png)

## Cloning Git Repo: 

```
mkdir -p ~/5g-deployment-lab/
git clone http://student:student@infra.5g-deployment.lab:3000/student/ztp-repository.git ~/5g-deployment-lab/ztp-repository/
git config --global user.email "you@example.com"
git config --global user.name "Your Name"
```

## Create Git Structure: 

```
cd ~/5g-deployment-lab/ztp-repository/
mkdir -p clusters/{site-group-1,site-group-1/secrets/sno2,site-group-1/secrets/sno3,site-group-1/sno2-extra-manifest,site-group-1/sno3-extra-manifest}
mkdir -p policies/{site-specific-policies,resources,configuration-version-2024-03-04,configuration-version-2024-03-04/source-crs,configuration-version-2024-03-04/manifests}
touch clusters/{site-group-1,site-group-1/secrets/sno2,site-group-1/secrets/sno3}/.gitkeep
touch policies/{site-specific-policies,resources,configuration-version-2024-03-04,configuration-version-2024-03-04/source-crs,configuration-version-2024-03-04/manifests}/.gitkeep
git add --all
git commit -m 'Initialized repo structure'
git push origin main
```

![image1](images/lab_build_0.png)
