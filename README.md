# Article CMS (FlaskWebProject)

This project is a Python web application built using Flask. The user can log in and out and create/edit articles. An article consists of a title, author, and body of text stored in an Azure SQL Server along with an image that is stored in Azure Blob Storage. You will also implement OAuth2 with Sign in with Microsoft using the `msal` library, along with app logging.

## Log In Credentials for FlaskWebProject

- Username: admin
- Password: pass

Or, once the MS Login button is implemented, it will automatically log into the `admin` account.

## Azure Settings of this Project
This page outlines all the settings and steps taken for this project.

1. ### Resource Group
- Resource Group Name: cms

2. ### SQL Database
- DB name: cms
- Server: cms95.database.windows.net
- DB region: us-east
- Admin login: cmsadmin
- Admin password: CMS4dmin
- Resource group: cms
- DB workload env: Development
- DB compute + storage: DTU - Basic
- Press the "Next: Networking" button, then select "Public Endpoint", and set both of the Firewall rules that appear to "Yes".
- Set everything else to default
- #### Run SQL queries in sql_scripts/ directory after completion, starting from the users table. The screenshots of the users and posts table are in the images folder.

3. ### Storage Account
- Resource group: cms
- Storage account name: gallery95
- Advanced - Allow enabling anonymous access on individual containers: Enable
- Advanced - Access tier: Cool
- Network access: Enable public access from all networks (the default)
- Create container named "images". Set its access level to Container.
- From Security + networking > Access keys:
    - Blob Storage key 1: Si89tSzDgGpkO7M87rK49a8Kn9soF36mK3VRGXLAi8EHFyo7icTM487Y87fTYbZqJteX0IQKFINg+AStJXIPjw==
    - Blob connection string 1: DefaultEndpointsProtocol=https;AccountName=gallery95;AccountKey=Si89tSzDgGpkO7M87rK49a8Kn9soF36mK3VRGXLAi8EHFyo7icTM487Y87fTYbZqJteX0IQKFINg+AStJXIPjw==;EndpointSuffix=core.windows.net
    - Blob storage key 2: 4BALqzjPQxK2Dz9gFMLbG69/Dwng16bSWKK4zFrzPIP0eHmHo5r3SOs8sxLC9Tf/5SwrttfSxKFJ+AStsT/KJg==
    - Blob connection string 2: DefaultEndpointsProtocol=https;AccountName=gallery95;AccountKey=4BALqzjPQxK2Dz9gFMLbG69/Dwng16bSWKK4zFrzPIP0eHmHo5r3SOs8sxLC9Tf/5SwrttfSxKFJ+AStsT/KJg==;EndpointSuffix=core.windows.net

4. ### Microsoft Entra ID

#### 4.1. App Registration
- Name: cmsEntraID
- Who can use? "Accounts in any organizational directory (Any Microsoft Entra ID tenant - Multitenant) and personal Microsoft accounts (e.g. Skype, Xbox)"
#### 4.2. Secret Creation
- Secret description: cmsSecret
- Secret Key: ff33137f-e95a-4ab8-822c-aa67611207b0
- Client Secret: 6up8Q~qYVjhDR25SVZ~BMsJ91q7qzjIErcwFSbqH
- Application (client) ID: 55f613be-63e0-4262-9360-e40690a94db4

5. ### Application
I picked option 2: Web App for setting up the application. The application and the OAuth2 login featurewas set up successfully.

#### 5.1. OPTION 1: Virtual Machine
- Name: vm-cms
- Authentication type: Password
- Username: cmsUser
- Password: cmsP4ssw0rd111
- Size: Standard B1ls (the cheapest)
- Ubuntu 24.04
- Select inbound ports: Choose 80, 443, and 22
#### Step 1. Connect to VM via SSH

#### Step 2. Setup GitHub SSH key

Run the following commands in the terminal:

- ssh-keygen -t ed25519 -C "yourgithubemail@example.com"
- Press enter on the next few prompts
- eval "$(ssh-agent -s)"
- ssh-add ~/.ssh/id_ed25519
- cat ~/.ssh/id_ed25519.pub
- Copy the output to https://github.com/settings/keys(opens in a new tab)
#### Step 3. Setup the website.

Setup HTTPS:

- sudo mkdir -p /etc/nginx/ssl
- sudo openssl req -x509 -nodes -days 365 -newkey rsa:2048 -keyout /etc/nginx/ssl/nginx.key -out /etc/nginx/ssl/nginx.crt
- Any values are accepted for the next questions. Here is an example:
    Country Name (2 letter code) [AU]:US
    State or Province Name (full name) [Some-State]:California
    Locality Name (eg, city) []:Palo Alto
    Organization Name (eg, company) [Internet Widgits Pty Ltd]:
    Organizational Unit Name (eg, section) []:
    Common Name (e.g. server FQDN or YOUR name) []:
    Email Address []:hello@example.com
Then, run the following commands:

- sudo curl https://packages.microsoft.com/config/ubuntu/22.04/prod.list | sudo tee /etc/apt/sources.list.d/mssql-release.list
- sudo apt update
- git clone [git@github.com:user/repo.git] web
- sudo apt -y update && sudo apt-get -y install nginx python3-venv unixodbc unixodbc-dev
- sudo ACCEPT_EULA=Y apt-get install -y msodbcsql17
- sudo unlink /etc/nginx/sites-enabled/default
- sudo vim /etc/nginx/sites-available/reverse-proxy.conf
- Paste this code into reverse-proxy.conf:
    server {
    listen 80;
    location / {
        proxy_pass https://localhost:5555;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection keep-alive;
        proxy_set_header Host $host;
        proxy_cache_bypass $http_upgrade;
    }
    }
    server {
    listen 443 ssl;
    ssl_certificate /etc/nginx/ssl/nginx.crt;
    ssl_certificate_key /etc/nginx/ssl/nginx.key;
    location / {
        proxy_pass https://localhost:5555;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection keep-alive;
        proxy_set_header Host $host;
        proxy_cache_bypass $http_upgrade;
    }
    }
- sudo ln -s /etc/nginx/sites-available/reverse-proxy.conf /etc/nginx/sites-enabled/reverse-proxy.conf
- sudo service nginx restart
- cd ~/web
- python3 -m venv venv
- . venv/bin/activate
- pip install --upgrade pip
- pip install -r requirements.txt
- Copy-paste the code from credentials.sh to the terminal
- python application.py

#### 5.2. OPTION 2: Web App (easier)
- Name: udacitycms95.azurewebsites.net
- Runtime stack: Python 3.10
- Pricing Plan: Free F1

After creation:

- Settings -> Environment variables - Add the following variables (sample values are included, replace them with your values):
    - BLOB_ACCOUNT: gallery95
    - BLOB_CONTAINER: images
    - BLOB_STORAGE_KEY: Si89tSzDgGpkO7M87rK49a8Kn9soF36mK3VRGXLAi8EHFyo7icTM487Y87fTYbZqJteX0IQKFINg+AStJXIPjw==
    - BLOB_CONNECTION_STRING: DefaultEndpointsProtocol=https;AccountName=gallery95;AccountKey=Si89tSzDgGpkO7M87rK49a8Kn9soF36mK3VRGXLAi8EHFyo7icTM487Y87fTYbZqJteX0IQKFINg+AStJXIPjw==;EndpointSuffix=core.windows.net
    - SQL_SERVER: cms95.database.windows.net
    - SQL_DATABASE: cms
    - SQL_USER_NAME: cmsadmin
    - SQL_PASSWORD: CMS4dmin
    - CLIENT_SECRET: 6up8Q~qYVjhDR25SVZ~BMsJ91q7qzjIErcwFSbqH
    - SECRET_KEY: ff33137f-e95a-4ab8-822c-aa67611207b0
    - CLIENT_ID: 55f613be-63e0-4262-9360-e40690a94db4
- #### Deployment Center
    - Source: GitHub
    - Pick the repo that contains the starter files.

6. ### Setting up OAuth2
At this point, your application should already be running. You should already be able to log in with username admin and password pass and you can create new posts or update existing ones.

The next part is getting the OAuth2 login to work.

- Go to Microsoft Entra ID > App Registrations, click on the App Registration created earlier, and then pick Authentication from the left sidebar.

#### 6.1. Microsoft Entra ID - Authentication - Add a Platform - Web
- Redirect URIs: https://udacitycms95-f0f6gpeza0gqagf8.canadacentral-01.azurewebsites.net/getAToken
- logout URL: https://udacitycms95-f0f6gpeza0gqagf8.canadacentral-01.azurewebsites.net/login

## Images Folder

This folder contains screenshots that proves that I completed various tasks throughout the project.

1. article cms.png is a screenshot from running the FlaskWebProject on Azure and prove that I was able to create a new entry. The Title, Author, and Body fields are populated to prove that the data is being retrieved from the Azure SQL Database while the image on the right proves that an image was uploaded and pulled from Azure Blob Storage.
2. azure portal resources group.png is a screenshot from the Azure Portal showing all of the contents of the Resource Group that I created. The resource group contains the following:
	- Storage Account
	- SQL Server
	- SQL Database
	- Resources related to deploying the app
3. sql dbo_users.png is a screenshot showing the users table and a query of data from the initial scripts.
4. sql dbo_posts.png is a screenshot showing the posts table and a query of data from the initial scripts.
4. blob endpoints.png is a screenshot showing the gallery95 blob endpoints for where images are sent for storage.
5. redirect URIs.png is a screenshot of the redirect URIs related to Microsoft authentication.
6. log-stream.png is a screenshot showing one potential form of logging with an "Invalid login attempt" and "admin logged in successfully", taken from the app's Log stream. 

## Dependencies

1. A free Azure account
2. A GitHub account
3. Python 3.10
4. Visual Studio 2019 Community Edition (Free)
5. The latest Azure CLI (helpful; not required - all actions can be done in the portal)

All Python dependencies are stored in the requirements.txt file. To install them, using Visual Studio 2019 Community Edition:
1. In the Solution Explorer, expand "Python Environments"
2. Right-click on "Python 3.10 (64-bit) (global default)" and select "Install from requirements.txt"

## Troubleshooting

- I had to update the pyodbc version in the requirements.txt file from pyodbc 4.0.28.0 to pyodbc 4.0.34 to resolve the incompability issue with the python version that prevented a successful deployment of the app.
- Mac users may need to install `unixodbc` as well as related drivers as shown below:
    ```bash
    brew install unixodbc
    ```
- Check [here](https://docs.microsoft.com/en-us/sql/connect/odbc/linux-mac/install-microsoft-odbc-driver-sql-server-macos?view=sql-server-ver15) to add SQL Server drivers for Mac.
