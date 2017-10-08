# Docker for RocketAgents

A decoupled Docker project for the RocketAgents dev environment.

### What's included

- PHP 5.6 (with APC, XDebug, and OpCache modules)
- PHP 7.1 (with XDebug, and OpCache modules)
- MySQL 5.6
- Apache 2.2

### Required Software

- Docker engine - https://docs.docker.com/engine/installation/

### Getting Started

1. Checkout the Rocket Docker project
		
        git clone git@github.com:rshugs/rocket-docker.git

2. Create a symbolic link from the main RocketAgents project into the RocketAgents Docker project

	ln -s /path/from/rocketagents rocket-docker/rocketagents

3. Copy the default configuration files affixed with ".orig"

        cp php56/php.ini.orig php56/php.ini
        cp php56/vhost.conf.orig php56/vhost.conf
        cp php71/php.ini.orig php71/php.ini
        cp php71/vhost.conf.orig php71/vhost.conf

4. Make sure the docker engine is running then build the containers. This can take a while.

	docker-compose build

5. Once containers have been built, you can now run a PHP5.6 server or PHP7.1 server which is available at `127.0.0.1`. However, only one of these containers can be run at a given time as they both use port 80.

	    # Run a PHP5.6 server
	    docker-compose up php56

	    # Or a PHP 7.1 server
	    docker-compose up php71

6. After selecting which server to run and server runs successfully, check the list of containers then type the following command in your terminal
		
        docker-compose ps or 
        sudo docker-compose ps

7. A list of containers will be displayed in your terminal together with the status and ports. To access a container, run the following command

        docker exec -it <container-name> bash or
        sudo docker exec -it <container-name> bash

        # For PHP 7.1 Server
      
        docker exec -it rocketdocker_php_71_1 bash or`
        sudo docker exec -it rocketdocker_php_71_1 bash

8. After successfully accessing php71 container, make sure to clear up your cache by running the following command
 
        php app/console cache:clear or 
        rm -r app/cache/prod/
        
> Note: Make sure that the *prod/* directory is removed.

9. Before updating the application databases. Make sure to check following databases if exist - rets and rocketagents. If it exist run the following script 

        ./app/console doctrine:schema:update --force

10. If the databases mentioned above does not exist, access the rocketdocker_db_1 container by running the following command
			
	docker exec -it rocketdocker_db_1 bash or 
        sudo docker exec -it rocketdocker_db_1 bash

11.  As the RocketAgents system is dependent on the domain, it is necessary to modify the local hosts file and have the development domain point to the local host `127.0.0.1`.

       # /etc/hosts

       # Domain may vary depending on your mysql dump
       # For the default dump included, `housesnv.local` may be used
       127.0.0.1    housesnv.local

7. You should now be able to access [housesnv.local/app.php](http://housesnv.local/app.php) or [housesnv.local/app_dev.php](http://housesnv.local/app_dev.php) on your browser

### Notes

If you can't access the application, manually import the rets and rocketagents sql files to your databases by running the following commands: 

        docker exec -i rocketdocker_db_1 mysql -uroot -procketagents rets < <file path for rets.sql file>
        docker exec -i rocketdocker_db_1 mysql -uroot -procketagents rocketagents < <file path for rocketagents.sql file>
 
### Profiling with XDebug

XDebug is conditionally enabled and can be triggered by appending `XDEBUG_PROFILE=1` to the request URL e.g.

	GET http://housesnv.local/app.php?XDEBUG_PROFILE=1

This will generate a cachegrind file with data regarding the request made. To view this data in a human readable format, install QCacheGrind -- available for Linux and MacOS systems.

![image of qcachegrind visualization](/assets/qcachegrind.png)

### Notes on OpCache

OpCache is PHP's default bytecode cache which dramatically improves performance when turned on. A big downside to this, however, is that codebase changes will not take effect unless the web server is restarted -- effectively clearing the entire cache.

The OpCache is turned off by default for practical reasons as this docker env is primarily for development. To turn it on, simply change `opcache.enable=0` to `opcache.enable=1` in `php56/php.ini` and rebuild the web container.

	docker-compose build php56 && docker-compose up php56

