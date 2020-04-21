# S3Server QuickStart guide
This is a step by step guide to get S3Server ready for you on your system.
Before cloning, however, you have your VMs setup with specifications mentioned in [Virtual Machine](VIRTUAL_MACHINE.md).

## Accessing the code right way
(For phase 1) Latest code which is getting evolved, advancing and contributed is on the gerrit server.
Seagate contributor will be referencing, cloning and committing code to/from this [Gerrit server](http://gerrit.mero.colo.seagate.com:8080).

Following steps will make your access to server hassel free.
1. From here on all the steps needs to be followed as root user.
  * Set the root user password using `sudo passwd` and set the password.
  * Type `su -` and enter root password to switch in to a root user mode.
2. Create SSH Public Key
  * [SSH generation](https://git-scm.com/book/en/v2/Git-on-the-Server-Generating-Your-SSH-Public-Key) will make your key generation super easy. follow the instructions throughly.
3. Add SSH Public Key on [Gerrit server](http://gerrit.mero.colo.seagate.com:8080).
  * Log into the gerrit server with your seagate gid based credentials.
  * On right top corner you will see your name, open drop down menu by clicking and choose settings.
  * In the menu on left, click SSH Public Keys, and add your public key (which is generated in step one) right there.

WoW! :sparkles:
You are all set to fetch S3Server repo now. 

## Cloning S3Server Repository
Getting the main S3Server code on your system is straightforward.
1. `$ cd path/to/your/dev/directory`
2. `$ export GID=<your_seagate_GID>` # this will make subsequent steps easy to copy-paste :)
3. `$ git clone "ssh://g${GID}@gerrit.mero.colo.seagate.com:29418/s3server"` 
4. Enable some pre-commit hooks required before pushing your changes to remote.
  * `$ scp -p -P 29418 g${GID}@gerrit.mero.colo.seagate.com:hooks/commit-msg "s3server/.git/hooks/"`
    
    if permission denied, then do following
    
    `$ chmod 600 /root/.ssh/id_rsa`

5. `$ cd s3server`
6. `$ git submodule update --init --recursive && git status`  # this can take about 20 minutes to run

## Installing dependency
This is a one time initialization when we do clone the repository or there is a changes in dependent packages.

  * At some point during the execution the `init.sh` script will prompt for following password, enter those as mentioned below.
    * SSH password: `XYRATEX`
    * Enter new password for openldap rootDN:: `seagate`
    * Enter new password for openldap IAM admin:: `ldapadmin`

1. `$ cd ./scripts/env/dev`
2. `$ ./init.sh`, For some system `./init.sh` fails sometimes. If it is failing run `./upgrade-enablerepo.sh` and re run `./init.sh`. Refer below image of successful run of `./init.sh` where `failed` field should be zero.  # this can take 10-15 minutes to run

<p align="center"><img src="../../assets/images/init_script_output.PNG?raw=true"></p>

## Compilation and Running Unit Test
All following commands assumes that user is already into it's main source directory.
### Running Unit test and System test
1. Setup host system
  * `$ ./update-hosts.sh`
2. Following script by default will build the code, run the unit test and system test in your local system. Check for help to get more details.
  * `$ ./jenkins-build.sh`. If you face issue with clang-format, to install refer [here](MeroQuickStart.md#getting-git--gerit-to-work).
  Make sure output log has message as shown in below image to ensure successful run of system test in `./jenkins-build.sh`.
  
<p align="center"><img src="../../assets/images/jenkins_script_output.PNG?raw=true"></p>

### Testing using S3CLI
1. Installation and configuration
  * Make sure you have `easy_install` installed using `$ easy_install --version`. If it is not installed run the following command.
    * `$ yum install python-setuptools python-setuptools-devel`
  * Make sure you have `pip` installed using `$ pip --version`. If it is not installed run the following command.
    * `$ python --version`, if you don't have python version 2.6.5+ then install python.
    * `$ python3 --version`, if you don't have python3 version 3.3+ then install python3.
    * `$ easy_install pip`
  * Make sure S3Server and it's dependent services are running.
    * `$ ./jenkins-build.sh --skip_build --skip_tests` so that it will start S3Server and it's dependent services.
    * `$ pgrep S3`, it should list the `PID` of S3 processes running.
    * `$ pgrep mero`, it should list the `PID` of mero processes running.
  * Install aws client and it's plugin
    * `$ pip install awscli`
    * `$ pip install awscli-plugin-endpoint`
  * Generate aws access key id and aws secret key
    * `$ s3iamcli -h` to check help messages.
    * `$ s3iamcli CreateAccount -n < Account Name > -e < Email Id >` to create new user. Enter the ldap `User Id` and `password` as mentioned below. Along with other details it will generate `aws access key id` and `aws secret key` for new user, make sure you save those.
      * Enter Ldap User Id: `sgiamadmin`
      * Enter Ldap password: `ldapadmin`
  * Configure AWS
    * `$ aws configure` and enter the following details.
      * AWS Access Key ID [None]: < ACCESS KEY >
      * AWS Secret Access Key [None]: < SECRET KEY >
      * Default region name [None]: US
      * Default output format [None]: text
  * Set Endpoint
    * `$ aws configure set plugins.endpoint awscli_plugin_endpoint`
    * `$ aws configure set s3.endpoint_url http://s3.seagate.com`
    * `$ aws configure set s3api.endpoint_url http://s3.seagate.com`
  * Make sure aws config file has following content
    * `$ cat ~/.aws/config`
  ```
  [default]
  output = text
  region = US
  s3 =
      endpoint_url = http://s3.seagate.com
  s3api =
      endpoint_url = http://s3.seagate.com
  [plugins]
  endpoint = awscli_plugin_endpoint
  ```
  * Make sure aws credential file has your access key Id and secret key.
    * `$ cat ~/.aws/credentials`
2. Test cases
  * Make Bucket
    * `$ aws s3 mb s3://seagatebucket`, should be able to get following output on the screen
      * `make_bucket: seagatebucket`
  * List Bucket
    * `$ aws s3 ls`, should be able to see the bucket we have created.
  * Remove Bucket
    * `$ aws s3 rb s3://seagatebucket`, bucket should get remove and should not be seen if we do list bucket.
  * Copy local file to remote(PUT)
    * `$ aws s3 cp test_data s3://seagatebucket/`, will copy the local file test_data and test_data object will be created under bucket.
  * List Object
    * `$ aws s3 ls s3://seagatebucket`, will show the object named as test_data
  * Move local file to remote(PUT)
    * `$ aws s3 mv test_data s3://seagatebucket/`, will move the local file test_data and test_data object will be created under bucket.
  * Remove object 
    * `$ aws s3 rm  s3://seagatebucket/test_data`, the test_data object should be removed and it should not be seen if we do list object.

KABOOM!!!
  
## Running Jenkins / System tests

TODO

## Code reviews and commits

To follow step by step process refer [here](MeroQuickStart.md#Code-reviews-and-commits).

### You're all set & You're awesome

In case of any query feel free to write to our [SUPPORT](support.md).

Let's start without a delay to contribute to seagate's open source initiative and join this movement with us keeping a common goal of making data storage better, more efficient and more accessible.

Seagate welcomes You! :relaxed:
