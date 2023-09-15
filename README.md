# A Step-by-Step Guide on How to Setup Django on Ubuntu 22.04 and Configure Postgres, Nginx, and Gunicorn 

Django, a potent Python web framework, is a crucial tool for quickly and effectively launching your website or application. It offers a solid framework that simplifies the development process and shortens time-to-market.

Django places a strong focus on security and includes protection against widespread web vulnerabilities like cross-site scripting and SQL injection, providing a strong base for a safe application.

Django also offers a dependable and effective framework for launching Python websites and applications, enabling developers to concentrate on the special features and business logic of their applications.

When Postgres is used as the database backend, data integrity and scalability are guaranteed, allowing for the easy management of big datasets. As a reverse proxy and high-performance web server, Nginx efficiently handles incoming requests and optimizes the delivery of static resources. 

Gunicorn acts as the WSGI HTTP server, making it easier to deploy Django apps and offering parallelism to handle numerous connections at once. A safe and scalable solution for Django web applications is provided by this configuration, allowing developers to concentrate on creating their projects without worrying about the supporting infrastructure.

In this article, you will set up a PostgreSQL database to replace the default SQLite database, install and set up a few components on Ubuntu 22.04 to support and serve Django apps, and then configure the Gunicorn application server to communicate with your applications.

Last but not least, you'll configure Nginx to act as a reverse proxy to Gunicorn, enabling you to use the application server's security and performance advantages.

Please note, that this guide requires you to have an Ubuntu 22.04 server configured, a non-root user with sudo capabilities set up, and a firewall turned on in order to finish this setup. If you haven’t set up your server yet. Use our step-by-step guide on how to set up a server on Ubuntu 22.04.


# Step 1: Update System Packages

The first step is to update the local apt package to reflect both new packages that have just entered the repositories and out-of-date ones that need to be upgraded. 
```
$ sudo apt update
$ sudo apt upgrade
```


## Step 2: Install Dependencies

Next, get everything you require from the Ubuntu repository and install them. This however will depend on which version of Python you’re using. It is highly advised that you select the latest Python version, Python 3 if you’re creating new projects.

If you’re using Django with Python 3, run the following code;

```
$ sudo apt install python3-pip python3-dev libpq-dev postgresql postgresql-contrib nginx curl
```


## Step 3: Create a PostgreSQL Database and User

In this step, you’ll be creating a PostgreSQL database and a database user which are vital in running your Django app. To create a PostgreSQL database and user, start by gaining access to the PostgreSQL server. Use sudo and the -u option to enter an interactive Postgres session via the following command line,
```
$ sudo -u postgres psql
```

By running the command, you’ll receive a PostgreSQL prompt to set up your database. Once inside the PostgreSQL prompt, use the **"CREATE DATABASE"** to provide your database's chosen name.

``
prostgres=# CREATE DATABASE yourproject;
``

Remember to include a semicolon at the end of your command line on every Postgres statement, most people experience issues with this.

Next, create a user and provide an authentication password using the **"CREATE USER"** command.

``
prostgres=# CREATE USER yourprojectuser WITH PASSWORD 'password';
``

Output

``
CREATE ROLE
``

To speed up database operations, alter a few of the user's connection parameters that you created. This will eliminate the need to query and set the right settings each time a connection is made.

Django requires UTF-8 as the default encoding. So, you need to set that here by using the following command line;

```
prostgres=# ALTER ROLE yourprojectuser SET client_encoding TO 'utf8';
```
Output

``
ALTER ROLE
``
The next thing you need to do is to configure ‘read committed’ to block reads from uncommitted transactions as the default transaction isolation technique.

```
prostgres=# ALTER ROLE yourprojectuser SET default_transaction_isolation TO 'read committed';
```
Output

``
ALTER ROLE
``

Then set the appropriate timezone because your Django projects will be configured by default to use UTC.

```
prostgres=# ALTER ROLE yourprojectuser SET timezone TO 'UTC';
```

Output

``
ALTER ROLE
``

Grant your new user access so they may manage your new database
```
prostgres=# GRANT ALL PRIVILEGES ON DATABASE yourproject TO yourprojectuser;
```
Output

``
GRANT
``

Using the **"GRANT"** command, gives the user the privileges they require, such as access to the newly constructed database or particular schemas inside it. Make sure that the new user has the necessary rights to successfully interact with the database before saving the modifications.

When finished, use the following command to leave the PostgreSQL prompt:

```
prostgres=# \q
```

Up to this point, Postgres is now successfully configured and Django can connect to it and manage its database data.


## Step 4: Create and Activate a Python Virtual Environment

For effective and well-organized work, a Python virtual environment must be created and activated. You can isolate the dependencies for your project and make sure they don't conflict with other Python installs on your machine by building a virtual environment. This lowers the possibility of version conflicts by enabling you to manage and control the particular versions of packages and libraries required for your project. 

Additionally, because they offer a constant and predictable environment across various devices, virtual environments make cooperation simple. By turning on a virtual environment, you may quickly move between many projects, each of which has its own set of dependencies, increasing development productivity and preserving code integrity.

To create and activate a Python virtual environment, you’ll need to use **virtualenv** command. Start by installing this with pip. Enable it by installing it with pip.

On Python 3 version, upgrade pip:

```
$ sudo -H pip3 install --upgrade pip
```

Then install virtualenv package:
```
$ sudo -H pip3 install virtualenv
```


After installing virtualenv you can now proceed with creating your project. Create the directory first, then save your project files there. Actually,  you can give your directory any name you want. But in this case, we'll call it yourdirectory.

```
$ mkdir ~/yourdirectory
```

The next thing you’ll need to do is to move into the directory you just created.

```
$ cd ~/yourdirectory
```

Next, inside your project directory, create a Python virtual environment.

```
$ virtualenv yourenv

```

In your **yourdirectory**, this will create a directory called **yourenv**. It will set up a local copy of pip and a local copy of Python inside. This can be used to set up a separate Python environment for your project.

Once the virtual environment is created, you need to activate it before installing the Python required for your project by doing the following;

```
$ source yourenv/bin/activate
```

The command prompt will change after activation to show that you are now working in the virtual environment. I will look like this **(yourenv)user@host:~/yourdirectory$**.

Utilizing a virtual environment allows you to maintain the independence of your project's dependencies and prevent conflicts with other Python projects or system-level packages. It enables you to keep a pristine and independent development environment tailored to your project.

You may now use pip to install any required Python packages or libraries required for your project such as Django, Gunicorn, and the psycopg2 PostgreSQL adaptor using the following command.

```
(yourenv) $ pip install django gunicorn psycopg2-binary

```

Once the necessary packages have been installed, you can begin working on your project in the virtual environment. Run any necessary Python programs and instructions.


Since you already have Python components installed, you can create your Django project. Navigate to the project directory you created earlier, **‘yourdirectory’**, and make Django install the files there. This will create a new directory with the project structure and necessary files.

```
(yourenv) $ django-admin.py startproject yourproject ~/yourdirectory

```

**Note:** If you are having problems, try using **django-admin** and not **django-admin.py**

So far, your project directory should contain the following content;

``
~/yourdirectory/manage.py:`` A Django project management script.

``
~/yourdirectory/yourproject/: ``The Django project package. This will contain the __init__.py, settings.py, urls.py, and wsgi.py files.

``
~/yourdirectory/yourenv/: ``The virtual environment directory you created earlier.


With your newly created Django project now, the next step is to configure the project settings.

Navigate into the project directory by running cd and type the project name you gave your project. Open the settings.py file located inside the project directory with a text editor. In this case, we’re going to use nano.

```
(yourenv) $ nano ~/yourdirectory/yourproject/settings.py
```


The Django project's behavior and attributes are controlled by a number of options in the settings file. You must determine the precise parameter you wish to change and edit its value before you can change the project settings.

By modifying the project parameters, you may tailor the behavior of your Django application to meet your unique needs and increase its flexibility, effectiveness, and security.

Start to modify your Django project settings by first locating the **ALLOWED_HOSTS** configuration option. Add the host names or IP addresses associated with your Django server. For example: **['example.com', '192.158.1.37']**. Ensure each item is listed in quotations with your entries separated by a comma.

The next step is to configure your project’s database. Navigate to a section labeled **DATABASES** within settings.py. Specify and configure the details of your database settings, such as the engine, name, user, password, and host.

With the information from your PostgreSQL database, update the parameters. After that, instruct Django to make use of the psycopg2 adaptor you installed via pip. Additionally, you must declare that the database is located on the local computer and supply the database name, the username of the database user you just created, and the password for that user. You can allow an empty string for the **PORT** setting.

```
DATABASES = {
    'default': {
        'ENGINE': 'django.db.backends.postgresql_psycopg2',
        'NAME': 'yourproject',
        'USER': 'yourprojectuser',
        'PASSWORD': 'password',
        'HOST': 'localhost',
        'PORT': ' ',
    }
}
```

Then, at the file's end, add a setting specifying the location for the static files. This is required in order for Nginx to handle requests for these things. The next line instructs Django to put them in the base project directory, the **"static"** directory.

``
STATIC_URL = '/static/'
STATIC_ROOT = os.path.join(BASE_DIR, 'static/')
``

After making the necessary adjustments, save and close the settings.py file. Press CTRL + X, followed by Y and ENTER if you’re working on nano.

That's it! Your Django project's settings have been adjusted



## Step 5: Completing Initial Project Setup

The next step is to migrate the original database information to your PostgreSQL database. You can do this through the following;

**Point to note:** 
Where manage.py is not working, try using python3 manage.py 

```
(yourenv) $ ~/yourdirectory/manage.py makemigrations
```

 Then,

```
(yourenv) $ ~/yourdirectory/manage.py migrate

```

Then, create a super user to act as an admin for your project.

```
(yourenv) $ ~/yourdirectory/manage.py createsuperuser
```

Now, you need to fill in your preferred username, enter your email address, then type and confirm a password.

Run the following command to gather all static content into the directory location you specified:

```
(yourenv) $ ~/yourdirectory/manage.py collectstatic

```

After that, the static files will be put in a directory within your project directory called static.

If you followed the original server setup instructions, your server should be protected by a **UFW firewall**. You must grant access to the port you'll be using in order to test the development server. Create an exception for port 8000 in this situation.

```
(yourenv) $ sudo ufw allow 8000

```
At this point, you can launch the Django development server, allowing you to test your project using the following command. 

```
(yourenv) $ ~/yourdirectory/manage.py runserver 0.0.0.0:8000
```

Now, navigate to your browser and type your server’s domain name or IP address followed by :8000:

As follows, **http://server_domain_or_IP:8000** 

Your screen should be showing the default Django index page.





You will be prompted for the administrator username and password you created with the createsuperuser command if you add /admin to the end of the URL in the address bar.

You can access the standard Django administrative page after authenticating. So, key in the username and password you created during createsuperuser section.

Press CTRL + C in the terminal window to stop the development server once you’re done.



## Step 6: Evaluation of Gunicorn's Project-Serving Capability

With your project’s virtual environment still active, evaluate if Gunicorn can serve your project by first moving into your project directory.

```
(yourenv) $ cd ~/yourdirectory
```

Next, you need to run Gunicorn here to load the WSGI module for the project.

```
(yourenv) $ gunicorn --bind 0.0.0.0:8000 yourproject.wsgi
```

By doing this, Gunicorn will launch using the same interface as the Django development server. You can try the application once more by going back.


**Point to Note:**
Gunicorn acts as an interface to your program, converting HTTP client requests into Python calls that your program can handle. The relative directory path to your application's entry point, the wsgi.py file in Django, was supplied to Gunicorn using the module syntax of Python.

Evaluating Gunicorn's performance and dependability as a web server for the particular application is part of testing its suitability for the project. This can be accomplished by modeling different user situations and monitoring variables like response time, throughput, and resource usage. 

Furthermore, stress testing can be done to find out the server's capacity under heavy pressure. Any bottlenecks or problems can be found and fixed by reviewing server logs and keeping an eye on system metrics, ensuring that Gunicorn is able to serve the project optimally and steadily.

Now you can exit the session. To terminate Gunicorn once testing is complete, hit **CTRL + C** in the terminal window.

You can now disable your virtual environment after completing the configuration of your Django application by the following;

```
(yourenv) $ deactivate
```

After deactivation of your project’s virtual environment, you will no longer see the virtual environment indicator in your prompt.


## Step 7: Create a Systemd Service for Gunicorn

After enabling Gunicorn to interact with your Django application, there is a need to create systemd service and socket files that ensure that Gunicorn boots along with the system and restarts it when something goes wrong.

Systemd has log in and monitoring features that let you keep track of and evaluate Gunicorn's performance as well as effectively fix any problems. The deployment and scaling of Gunicorn are made simpler by systemd integration, making it simpler to manage many instances and transfer the workload across several servers or environments. Overall, Gunicorn's operation and upkeep are simplified by leveraging systemd socket and service files, which improves stability and effectiveness.

Let’s now create and open a systemd socket file for Gunicorn with sudo privileges;

```
$ sudo nano /etc/systemd/system/gunicorn.socket

```
Create three sections there: Unit to describe the socket; Socket to specify its location; and Install to ensure that the socket is created at the appropriate time.

```
[Unit]
Description=gunicorn socket

[Socket]
ListenStream=/run/gunicorn.sock

[Install]
WantedBy=sockets.target

```

Save and exit the file once you’re done.


Next, open your chosen text editor and create a systemd service file for Gunicorn with sudo permissions. With the exception of the extension, the service filename should coincide with the socket filename:

```
$ sudo nano /etc/systemd/system/gunicorn.service

```

Then, add the following configuration: 

```
[Unit]
Description=gunicorn daemon
Requires=gunicorn.socket
After=network.target

[Service]
User=james
Group=www-data
WorkingDirectory=/home/james/yourdirectory
ExecStart=/home/james/yourdirectory/yourenv/bin/gunicorn \
          --access-logfile - \
          --workers 3 \
          --bind unix:/run/gunicorn.sock \
          yourproject.wsgi:application

[Install]
WantedBy=multi-user.target

```

Your systemd service file is now set. Save and exit the document.

Now, launch the Gunicorn socket. The socket file will be created and located at **/run/gunicorn.sock**.

```
$ sudo systemctl start gunicorn.socket

```

Then enable the Gunicorn Socket so that systemd will launch the gunicorn.service immediately to handle connections to that socket.

```
$ sudo systemctl enable gunicorn.socket

```

By looking for the socket file, you can now verify that the process was successful.


## Step 8: Check for Gunicorn Socket File

Checking the Gunicorn socket file helps to confirm the existence of the socket file and ensure Gunicorn is set up properly to use the provided socket file for handling incoming requests.
Now, check your Gunicorn status to verify if the process worked just fine using the following;

```
$ sudo systemctl status gunicorn.socket

```
Your status should be similar to this;

```
● gunicorn.socket - gunicorn socket
     Loaded: loaded (/etc/systemd/system/gunicorn.socket; enabled; vendor prese>
     Active: active (listening) since Thu 2023-05-11 08:21:48 UTC; 1s ago
   Triggers: ● gunicorn.service
     Listen: /run/gunicorn.sock (Stream)
      Tasks: 0 (limit: 1136)
     Memory: 0B
     CGroup: /system.slice/gunicorn.socket

May 18 12:34:12 gunicorn systemd[1]: Listening on gunicorn socket.

```


Next, confirm whether you have gunicorn.sock file within the **/run directory**. The socket file should be specified here.

```
$ file /run/gunicorn.sock
```

Results

    
     /run/gunicorn.sock: socket
    

If the socket file doesn't exist, it means that Gunicorn is not currently running or hasn't created the socket file yet. If the gunicorn.sock file is missing from the directory. Run the following commands to check the Gunicorn socket's logs:

``` 
$ sudo journalctl -u gunicorn.socket

```

Look for the line that contains, **/etc/systemd/system/gunicorn.socket** file to check and fix problems before continuing.


## Step 9: Text Socket Activation Status

The gunicorn.service won't be active just yet if you only started the gunicorn.socket unit. This is because it hasn't received any connections. Verify the situation using the following:

```
$ sudo systemctl status gunicorn
```

Results

```
● gunicorn.service - gunicorn daemon
   Loaded: loaded (/etc/systemd/system/gunicorn.service; disabled; vendor preset: enabled)
   Active: inactive (dead)
```

Send a connection to the socket using curl to verify the socket activation process.

```
$ curl --unix-socket /run/gunicorn.sock localhost

```

At this point, you should be seeing an HTML output on your screen. This shows that Gunicorn was started correctly and can serve your Django project.

Again confirm whether Gunicorn service is working by verifying the status once more.

```
$ sudo systemctl status gunicorn
```

Results

```
●  gunicorn.service - gunicorn daemon
   Loaded: loaded (/etc/systemd/system/gunicorn.service; disabled; vendor preset
   Active: active (running) since Fri 2023-05-19 14:21:12 UTC; 5s ago
 Main PID: 11002 (gunicorn)
    Tasks: 4 (limit: 1151)
   CGroup: /system.slice/gunicorn.service
          ├─11002 /home/james/yourdirectory/yourenv/bin/python /home/james/
         ├─11018  /home/james/yourdirectory/yourenv/bin/python /home/james/
         ├─11020  /home/james/yourdirectory/yourenv/bin/python /home/james/
         └─11022  /home/james/yourdirectory/yourenv/bin/python /home/james/
. . .

```

Check the logs for more information if the output of curl or systemctl status indicates there is a problem. Use this;

```
$ sudo journalctl -u gunicorn

```

Look for issues in the **/etc/systemd/system/gunicorn.service** file. Reload the daemon to allow the daemon to read the service definition again if you make modifications to the /etc/systemd/system/gunicorn.service file.

```
$ sudo systemctl daemon-reload
```


Next, start the Gunicorn process again;

```
$ sudo systemctl restart gunicorn 

```

Always ensure if any of these problems arise, fix them before moving on.


## Step 10: Setting Up Nginx to Proxy Pass to Gunicorn

You need to make changes to the Nginx configuration file to configure Nginx to proxy pass to Gunicorn. You can do this by opening a new server block, **sites-available** in Nginx using the following;

```
$ sudo nano /etc/nginx/sites-available/yourproject
```

When inside, create the server block in the file that matches the server name or IP address you want to use. Indicate that your server's domain name or IP address should be used to invoke this block's response and that it should listen on standard port 80.


Then tell Nginx to disregard any favicon-related issues. Additionally, specify the location of the static assets you gathered and stored in the **/yourdirectory/static** directory. You can create a location block to match those requests because all of these files use the standard URI prefix "/static":

Lastly, define a location / {} block to match all subsequent requests. Include the default proxy_params file from your Nginx installation inside of this directory, and then direct traffic to the Gunicorn socket

``
(path)  /etc/nginx/sites-available/yourproject
``

```
server {
    listen 80;
    server_name server_domain_or_IP;
  
location = /favicon.ico { access_log off; log_not_found off; }
    location /static/ {
        root /home/james/yourdirectory;
    }
location / {
        include proxy_params;
        proxy_pass http://unix:/run/gunicorn.sock;
    } 
}

```

When you're done, save and exit the file. Now that the file is linked to the sites-enabled directory, you can enable it:

```
$ sudo ln -s /etc/nginx/sites-available/yourproject /etc/nginx/sites-enabled
```

Then verify the syntax errors of your Nginx configuration using the following:

```
$ sudo nginx -t
```

Nginx should be restarted if there are no issues reported. Well, go ahead and restart Nginx.

```
$ sudo systemctl restart nginx
```

Then change your firewall settings by removing the rule to open port 8000 as you no longer require access to the development server:

```
$ sudo ufw delete allow 8000
```

Then, use the following command to enable regular traffic on ports 80 and 443 – enabling HTTP and HTTPS connections, respectively:

```
$ sudo ufw allow 'Nginx Full'
```

Up to this point, you should access your application by going to the domain or IP address of your server.



## Addressing Typical Gunicorn and Nginx Problems

Typical problems may occur in the configuration or functioning of Nginx and Gunicorn on a Django application. Some of the common problems include;

 - ### Nginx displays the default page instead of proxying to your application
When the default Nginx page appears instead of your Django application, Nginx is not set up correctly to serve your Django application. Here are some measures you can take to investigate and resolve the problem:

#### – Verify Nginx settings
Make sure the request routing to your Django application is configured appropriately in your Nginx configuration file. The /etc/nginx/sites-available/yourproject directory is normally where you may find the configuration file. You can examine the file's content by opening it with a text editor.

Verify that the domain or IP address you use to access your Django application matches the server_name directive in the Nginx setup.

Make sure the location block inside the server block is correctly set up to send requests to the Gunicorn or WSGI server running your Django application. It must contain pertinent directives and settings like proxy_pass.

#### – Check the Django application
Verify the proper operation of your Django application. Make that the WSGI or Gunicorn server is up and receiving requests.

Verify that the correct host and port are selected for your Django application to listen on. Gunicorn by default listens on localhost:8000, whereas WSGI can utilize other setups.

To make sure your Django application is functioning properly, check that it can be accessed directly using the host and port indicated in the settings of the Django application.

#### – Restart the Django application server and Nginx
You have to restart the Nginx service after making any changes to the Nginx configuration file in order for the changes to take effect. To restart Nginx, use the command:

```
$ sudo service nginx restart
```

To confirm that your Django application server (Gunicorn or WSGI) is using the changed configuration, restart the server as well.

#### – Verify the firewall's settings
Ensure that the ports utilized by Nginx and your Django application can receive incoming connections in your firewall settings. Nginx typically uses port 80 for HTTP and port 443 for HTTPS.

#### – Check your DNS settings
Check that the DNS settings are appropriately setup to point to the server where Nginx and your Django application are located if you're using a domain name to access your Django application.

#### – Examine the application logs and Nginx
Look through the Nginx error logs in /var/log/nginx/error.log for any particular error messages that might point to the problem's origin. Look for any problems or warnings in your Django application server's logs (Gunicorn or WSGI) that can help you solve the issue.

You should be able to pinpoint and fix the problem with Nginx not successfully delivering your Django application by using the above-listed techniques.

# Conclusion
As a result of following this guide, you have successfully installed Django, Postgres, Nginx, and Gunicorn on Ubuntu 22.04, creating a stable and effective environment for web development. The robust foundation of Django makes it simple for developers to create and launch sophisticated web applications. 

Although the instructions in this article are particular to Ubuntu 22.04, it's crucial to remember that the same procedure may be applied to other operating systems and versions with minor adjustments. In order to guarantee the stability and security of the entire setup, it is also essential to keep up with the most recent security patches and adhere to best practices. 

The performance of the stack should be regularly monitored and optimized to improve user experience. Using this comprehensive setup, developers can create high-quality, scalable online applications by combining the strengths of Django, Postgres, Nginx, and Gunicorn.
