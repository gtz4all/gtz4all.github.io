## Lambda Layers
Lambda layers contain the dependencies needed by any particular Lambda function. For example, a Lambda function written in Python might utilize extension Python modules, and these modules would be made available to the code via a Layer that contained on or more modules and any other objects upon which the required modules were dependent. Here's a walk though of adding one module - ldap3 - and then calling into that module from the Lambda function.

1. Create a python virtual environment on some machine (not in Lambda). For example - use the command python-m venv venv --prompt anLdapProject from the root of your project directory.
2. Activate the environment with .\venv\Scripts\activate(.bat/.ps1)on Windows, or source venv/bin/activate on Mac/Linux shell, from the root of your project directory.
3. Install the module you need - eg pip install ldap3. This will install the module and its dependencies. Take note of what it installs.
4. Repeat step 2 for as many modules as are needed by the Lambda function.
* This will create a subdirectory .\venv\Lib\site-packages containing material you need to compress and upload to the AWS Lambda context.
5. Copy the site-packages directory to another temporary working location. Rename it python.
Remove directories within the python directory that do not specifically pertain to the module you installed and its dependencies. Typically this will mean removing pip, setuptools etc.
6. Compress the python folder to a .zip file with an appropriate name reflecting the modules or the project you are building.
7. Upload the compressed file as a Layer that will be available to Lambda functions.
8. Associate that layer with your Lambda function. You can associate up to 5 layers totalling no more than 250 Mb of supporting materials.
9. Add the related import statements to your code and test that the import statements execute successfully
10. Add the function code for which you imported the modules and incrementally test as you progress.

## Python Virtual Environments - git - AWS Lambda - Why and How

This will only consider Python3's venv functionality. In this blog venv will be short for virtual environment, and also is the name of a python subcommand and in my projects is also the name of the subdirectory holding the venv. From python.org -

Virtual Environments allow you to manage separate package installations for different projects. They essentially allow you to create a “virtual” isolated Python installation and install packages into that virtual installation. When you switch projects, you can simply create a new virtual environment and not have to worry about breaking the packages installed in the other environments. It is always recommended to use a virtual environment while developing Python applications.

All but the most simple python solutions depend on site packages - packages that are beyond what is delivered in the standard packages for any version of Python. Anything we do against AWS objects requires the boto3 package.

Very common site package examples we use are boto3, jupyterlab, pyodbc, requests. Others we may use are django, djangorestframework, pandas, pendulum, ldap.

We need virtual environments because some package versions are dependent on or can conflict with other python package versions.

We also benefit from creating a clean environment that contains all the additional packages that we need and only the additional packages we need for a particular application. We can share the list of packages and their versions through a requirements.txt (or other) file, and the requirements file can be part of the repository of the application/project. The requirements file can be used to add all the site packages to a virtual environment on another machine - say when transitioning from DEV to TEST to PROD - and further we can make specific requirement adjustments to each environment.

A virtual environment should not be part of the git repository, but the requirements file that defines the packages of the virtual environment should be part of the repository. Therefore we need to add at least one line to our .gitignore file(s) to hide the virtual environment.

When creating AWS Lambda functions we can incorporate the packaged defined to a virtual environment into a Lambda layer - and on our BCH Cloud Confluence page Lambda Layers describes the conversion from a virtual environment to a Layer.

#### Life Cycle of a Virtual Environment
```bash
# Starting from having no project locally and the shell of a project or a full blown project on our GitLab.
# Sitting locally in the folder above where you want the project. For example in your repos folder
# don't use this example - not ready for sharing - feel free to look at it on GitLab
git clone git@ccts3.aws.chboston.org:Barney.Carney/movie-rater-api.git
cd movie-rater-api
python -m venv venv --prompt movra # venv is the python subcommand, venv is the directory created for the venv, and movra is the prompt when venv is activated.
# Now a virtual environment exists in the venv directory
# Activate the venv on Windoze
.\venv\Scripts\activate.bat # see the prompt change to: (movra) C:\...\movie-rater-api>
# Activate the venv on Linux/Unix/Mac
source venv/bin/activate # see the prompt change to: (movra) [ec2-user@ip-100-80-208-232 movie-rater-api]$
# add python site package dependencies - which may start with a set of known requirements.
pip install -r requirements.txt # if you have a project that already is underway and has requirements
pip install boto3 # if you are starting the project and/or adding boto3 code to the project
pip install jupyterlab # if you are using Jupyter Notebooks
# time goes by, you work on the project, you see you need to access a SQL database
pip install pyodbc
# take a look at your packages
pip list
# now I am working on another project in the same command session, so I want to deactivate my virtual environment
deactivate
pip list # should be different from the venv
# time goes by, you work on the project, you see you need to make https requests and get data back from a Rest API
.\venv\Scripts\activate.bat
pip install requests
# more time
pip install whatever-else-you-need
# capture your requirements as you go and certainly when you are finished
pip freeze > requirements.txt
```
https://docs.aws.amazon.com/lambda/latest/dg/python-package.html#python-package-venv
