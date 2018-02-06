# KRK_server Development and Deploy Guide

## 1. How to develop in Development Environment

### 1.1 Development environment

Currently, KRK_server is running with ruby-2.3 version and rails-5.0.
To continue developing, please install ruby-2.3 and rails-5.0 follow this guide on [Go Rails](https://gorails.com/setup/ubuntu/16.04).

### 1.2 Get newest code and Import master data

To develop KRK_server in development environment, please follow thoese steps:

**1.2.1 Pull code from remote repo**

```
git clone git@github.com:framgia/KRK_server.git
```
**1.2.2 Install dependence gem**

```
cd KRK_server
bundle install
```
**1.2.3 Import master data**

To setup for this step, please config Environment Variable for database:

```
# In your shell config file (e.g: ~/.bashrc or ~/.zshrc)
export DATABASE_USERNAME=your_mysql_username
export DATABASE_PASSWORD=your_mysql_password
```

This config will be used in config/database.yml

```
rake db:create
rake db:migrate
rake master_data:recreate_all_master_data
```

Then, you can run `rails s` and access to [http://localhost:3000](http://localhost:3000) to start coding.

**NOTE** :

1.After you do any features, please run `rspec spec/` to ensure that your code doesn't affect to other features.

2.Please view file `./.env_example` to see more environment variable examples (config for mailer, GMO account, ...) or view example [here](https://github.com/framgia/KRK_server/wiki/Environment-Variables).


## 2. How to deploy in Production Environment

This guide contains some steps to deploy a release in production mode.

### 2.1. Create tag

* Go to page: https://github.com/framgia/KRK_server/releases/new
* Put info in text boxes:


|Field | Description | Example|
|------|-------------|---------|
|Tag version |Required. Store version of a tag |v0.1-demo|
|Target |Required. Choose which branch will be deploy. It should be master branch |master|
|Release title |Required. Store title of this tag. It can be same name withtag version| v0.1 - 01/01/|
|Describe thisrelease |Optional. Store describe of this release, include date, time orsome reason why having this release |Hot fix bugs ...|

* Click to button `Pushlish release`

### 2.2. Update changelogs (Optional)

You should update change logs for each release to describe which changes are included in this release.
For example: [Ransack ChangeLog](https://github.com/activerecord-hackery/ransack/blob/master/CHANGELOG.md)

### 2.3. Backup Database

* SSH to your production machine.

Note: You have to ssh to Admin instance. (because 2 API intances only accept ssh requests from LB instance, not from your computer)

* Change user to deploy user

```
ssh -i key.pem user@ip_address
sudo su -
su - deploy
```
* Run backup database command

```
mysqldump -u$DATABASE_USERNAME -p$DATABASE_PASSWORD -h$DATABASE_HOSTNAMEkrk_production > backup_current_dattime.sql
```
This step requires database username and password in ENV variable. You can config it in `~/.env` file.

**NOTE** : All deploy scripts are stored in `/home/deploy/deploy_bin/` folder.

### 2.4. Deploy

* Firstly, you have to deploy in Admin Instance:

```
localdeploy tag $tag_version_which_you_created_above
```

* Secondly, you deploy on 2 API instances with this command:

```
deploy tag $tag_version_created_above
```

NOTE: All cronjobs, rake db:migrate will only run in admin instance.

### 2.5. Config AutoScale

Go to Amazone EC2 console. For example:
[List instances](https://ap-northeast-1.console.aws.amazon.com/ec2/v2/home?region=ap-northeast-1#Instances:sort=instanceId)

**Create Image**

Click to `Instances` and choose an **API Instance**, click `Actions > Image > Create Image`

![](https://viblo.asia/uploads/aa184a3b-bab3-4f50-bc7c-cdbc6d8a1d6f.png)

**Fill Image description**

**NOTE:** Tick to `No reboot` option => Force machine no reboot when autoscaling process runs.

Fill info to Image_name and Image description. It may be datetime when you create this image
After that, click to `Create Image` button. You can go to `Image/AMIs` to see creating process.

**Create Launch Configurations for AutoScaling**

* Copy old launch configuration: You have to choose instance which have instance type is `t2.medium`.

![](https://viblo.asia/uploads/52aa39e8-ca25-4b1b-b8c2-db856d328dd0.png)

After that, all config of old launch configuration will be copied. You must go to first step:

* Choose AMI to change some info.

Search and choose Image which you have been created above. After that, click Select.

![](https://viblo.asia/uploads/b9b9227e-97bc-49da-9ba5-450db977af5c.png)

Click Next: Configure detail

* At step 3. Configure details: You change Name of this configuration. (May be datetime when you create)

* Click Next: Configure Security Group >>> Review >>> Create launch configuration >>> Tick to Accept Term >>> Create launch configuration.

**Config in Auto Scaling Groups**

Click to `Edit` button in Auto Scaling Group. In Launch Configuration, search and choose configuration which you created above. >>> CLick Save

![abc](https://i.imgur.com/BJE9o84.png)


=> It's end. Now your code has been publised in production!
