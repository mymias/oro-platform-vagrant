# -*- mode: ruby -*-
# vi: set ft=ruby :

# All Vagrant configuration is done below. The "2" in Vagrant.configure
# configures the configuration version (we support older styles for
# backwards compatibility). Please don't change it unless you know what
# you're doing.
Vagrant.configure("2") do |config|
  # The most common configuration options are documented and commented below.
  # For a complete reference, please see the online documentation at
  # https://docs.vagrantup.com.

  # Every Vagrant development environment requires a box. You can search for
  # boxes at https://atlas.hashicorp.com/search.
  config.vm.box = "ubuntu/xenial64"

  # Disable automatic box update checking. If you disable this, then
  # boxes will only be checked for updates when the user runs
  # `vagrant box outdated`. This is not recommended.
  # config.vm.box_check_update = false

  # Create a forwarded port mapping which allows access to a specific port
  # within the machine from a port on the host machine. In the example below,
  # accessing "localhost:8080" will access port 80 on the guest machine.
  config.vm.network "forwarded_port", guest: 80, host: 8000

  # Create a private network, which allows host-only access to the machine
  # using a specific IP.
  config.vm.network "private_network", ip: "192.168.33.10"

  # Create a public network, which generally matched to bridged network.
  # Bridged networks make the machine appear as another physical device on
  # your network.
  # config.vm.network "public_network"

  # Share an additional folder to the guest VM. The first argument is
  # the path on the host to the actual folder. The second argument is
  # the path on the guest to mount the folder. And the optional third
  # argument is a set of non-required options.
  config.vm.synced_folder ".", "/home/ubuntu/www", owner: "www-data", group: "www-data"

  # Provider-specific configuration so you can fine-tune various
  # backing providers for Vagrant. These expose provider-specific options.
  # Example for VirtualBox:
  #
  config.vm.provider "virtualbox" do |vb|
  #   # Display the VirtualBox GUI when booting the machine
  #   vb.gui = true
  #
  #   # Customize the amount of memory on the VM:
    vb.memory = "3072"
  end
  #
  # View the documentation for the provider you are using for more
  # information on available options.

  # Define a Vagrant Push strategy for pushing to Atlas. Other push strategies
  # such as FTP and Heroku are also available. See the documentation at
  # https://docs.vagrantup.com/v2/push/atlas.html for more information.
  # config.push.define "atlas" do |push|
  #   push.app = "YOUR_ATLAS_USERNAME/YOUR_APPLICATION_NAME"
  # end

  # Enable provisioning with a shell script. Additional provisioners such as
  # Puppet, Chef, Ansible, Salt, and Docker are also available. Please see the
  # documentation for more information about their specific syntax and use.
  config.vm.provision "shell", inline: <<-SHELL

	echo "*****************************************************"
	echo "************* Provision process started *************"
	echo "*****************************************************"

	# --------------------- Provision configuration ---------------------

	echo -e "\n******** Provision configuration ********\n"

	# --- Database settings ---

	DB_USER="dbuser"
	DB_PASSWORD="dbpass"
	DB_NAME="oro"
	DB_NAME_TEST="oro_test"

	# --- Oro application settings ---

	APP_HOST="localhost"
	APP_USER="admin"
	APP_PASSWORD="adminpass"
	APP_LOAD_DEMO_DATA="y"    # y | n

	# --- Git settings ---

	# If you don't want to download source files from scratch (for example if you in development of your
	# Oro-based app, with your own existing sources), just comment GIT_ variables below (by # sign)
	GIT_REPO="https://github.com/orocrm/platform-application.git"

	# If you don't want to checkout a particular tag or branch (just stay at the HEAD of master
	# branch), comment GIT_TAG variable below
	GIT_TAG="1.10.8"
	
	# --------------------- LEMP installation & configuration ---------------------

	echo -e "\n******** Step 1 of 2. LEMP installation & configuration ********\n"

  	# --- Preconfigure mysql root password ---

  	echo -e "\n*** Preconfigure mysql root password ***\n"
  	echo "mysql-server mysql-server/root_password password $DB_PASSWORD" | debconf-set-selections
  	echo "mysql-server mysql-server/root_password_again password $DB_PASSWORD" | debconf-set-selections
  	
  	# --- Main installations ---

  	echo -e "\n*** Apt-get update ***\n"
  	apt-get update
  	echo -e "\n*** Install git, nginx, php, mysql-client, mysql-server, nodejs ***\n"
  	apt-get install --assume-yes git nginx mysql-client mysql-server nodejs php php7.0-xml php7.0-intl php7.0-mysql php-mbstring php7.0-curl php7.0-gd php7.0-mcrypt php7.0-soap php7.0-tidy php7.0-zip

  	# --- DB installation tuning ---

  	echo -e "\n*** DB installation tuning ***\n"
  	mysql -uroot -p$DB_PASSWORD -e "CREATE DATABASE $DB_NAME"
  	mysql -uroot -p$DB_PASSWORD -e "grant all privileges on $DB_NAME.* to '$DB_USER'@'localhost' identified by '$DB_PASSWORD'"
  	mysql -uroot -p$DB_PASSWORD -e "CREATE DATABASE $DB_NAME_TEST"
  	mysql -uroot -p$DB_PASSWORD -e "grant all privileges on $DB_NAME_TEST.* to '$DB_USER'@'localhost' identified by '$DB_PASSWORD'"
  	
  	# --- Nginx site config ---

  	echo -e "\n*** Create nginx site config ***\n"
  	cat > /etc/nginx/sites-available/$APP_HOST <<____NGINXCONFIGTEMPLATE
        server {
            server_name $APP_HOST www.$APP_HOST;
            root  /home/ubuntu/www/web;

            location / {
                # try to serve file directly, fallback to app.php
                try_files \\$uri /app.php\\$is_args\\$args;
            }

            location ~ ^/(app|app_dev|config|install)\.php(/|$) {
                fastcgi_pass unix:/run/php/php7.0-fpm.sock;
                fastcgi_split_path_info ^(.+\.php)(/.*)$;
                include fastcgi_params;
                fastcgi_param SCRIPT_FILENAME \\$document_root\\$fastcgi_script_name;
                fastcgi_param HTTPS off;
            }

            error_log /var/log/nginx/${APP_HOST}_error.log;
            access_log /var/log/nginx/${APP_HOST}_access.log;
        }
____NGINXCONFIGTEMPLATE
  	ln -s /etc/nginx/sites-available/$APP_HOST /etc/nginx/sites-enabled/$APP_HOST
  	service nginx restart
  	
  	# --- Add ubuntu user to www-data group (for composer) ---

   	echo -e "\n*** Add 'ubuntu' user to 'www-data' group ***\n"
 	usermod -a -G www-data ubuntu
 	
 	# ---  Configure php-fpm ---

   	echo -e "\n*** Configure php-fpm ***\n"
 	sed -i 's/;listen.allowed_clients/listen.allowed_clients/g' /etc/php/7.0/fpm/pool.d/www.conf
 	sed -i 's/;pm.max_requests/pm.max_requests/g' /etc/php/7.0/fpm/pool.d/www.conf
 	
 	# --- Set recommended PHP.INI settings ---

   	echo -e "\n*** Set php.ini settings ***\n"
 	# FPM php.ini
 	sed -i 's/;date.timezone =/date.timezone = Europe\\/London/g' /etc/php/7.0/fpm/php.ini
 	sed -i 's/memory_limit = [0-9MG]*/memory_limit = 1G/g' /etc/php/7.0/fpm/php.ini
 	sed -i 's/max_execution_time = [0-9]*/max_execution_time = 600/g' /etc/php/7.0/fpm/php.ini
 	sed -i 's/;opcache.enable=0/opcache.enable=1/g' /etc/php/7.0/fpm/php.ini
 	sed -i 's/;opcache.memory_consumption=[0-9]*/opcache.memory_consumption=256/g' /etc/php/7.0/fpm/php.ini
 	sed -i 's/;opcache.interned_strings_buffer=[0-9]*/opcache.interned_strings_buffer=8/g' /etc/php/7.0/fpm/php.ini
 	sed -i 's/;opcache.max_accelerated_files=[0-9]*/opcache.max_accelerated_files=11000/g' /etc/php/7.0/fpm/php.ini
 	sed -i 's/;opcache.fast_shutdown=[01]*/opcache.fast_shutdown=1/g' /etc/php/7.0/fpm/php.ini
 	# CLI php.ini
 	sed -i 's/;date.timezone =/date.timezone = Europe\\/London/g' /etc/php/7.0/cli/php.ini
 	sed -i 's/max_execution_time = [0-9]*/max_execution_time = 600/g' /etc/php/7.0/cli/php.ini
 	sed -i 's/;opcache.enable=0/opcache.enable=1/g' /etc/php/7.0/cli/php.ini
 	sed -i 's/;opcache.memory_consumption=[0-9]*/opcache.memory_consumption=256/g' /etc/php/7.0/cli/php.ini
 	sed -i 's/;opcache.interned_strings_buffer=[0-9]*/opcache.interned_strings_buffer=8/g' /etc/php/7.0/cli/php.ini
 	sed -i 's/;opcache.max_accelerated_files=[0-9]*/opcache.max_accelerated_files=11000/g' /etc/php/7.0/cli/php.ini
 	sed -i 's/;opcache.fast_shutdown=[01]*/opcache.fast_shutdown=1/g' /etc/php/7.0/cli/php.ini
	# "save comments", "load comments" for lexical parser for doctrine?

 	# --- Set recommended MySQL settings ---

   	echo -e "\n*** Configure MySQL ***\n"
	echo "innodb_file_per_table = 0" >> /etc/mysql/my.cnf
	echo "wait_timeout = 28800" >> /etc/mysql/my.cnf

   	echo -e "\n*** LEMP installation finished ***\n"

	# --------------------- Oro application installation ---------------------

	echo -e "\n******** Step 2 of 2. Oro application installation ********\n"

  	cd www

 	# --- Get application source files (if needed) ---

 	if [ -n $GIT_REPO ]; then
   	    echo -e "\n*** Get source files ***\n"
   	    git clone $GIT_REPO tmpdir

   	    # Checkout a particular tag or branch (if needed)
   	    if [ -n $GIT_TAG ]; then
   	        cd tmpdir
   	        git checkout $GIT_TAG
   	        cd ..
   	    fi

   	    # Move source files to working (parent) directory from tmpdir
   	    # Move regular files
   	    mv tmpdir/* .
   	    # Move hidden files and dirs (.gitignore, .travis, etc)
   	    rm -rf tmpdir/.git      # remove not needed hidden files and dirs
   	    mv -f tmpdir/.[!.]* .
   	    # Cleanup
   	    rm -rf tmpdir

   	    # add /.vagrant to .gitignore
   	    echo ".vagrant" >> .gitignore
 	fi

 	# --- Speedup Symfony on Vagrant ---

 	# See http://www.whitewashing.de/2013/08/19/speedup_symfony2_on_vagrant_boxes.html
 	echo -e "\n*** Speedup Symfony on Vagrant ***\n"
 	rm -rf app/cache app/logs
 	ln -s /dev/shm app/cache
 	ln -s /dev/shm app/logs
 	
 	# --- Configure app/config/parameters.yml ---

 	# (to prevent composer interactive dialog)
 	echo -e "\n*** Configure app/config/parameters.yml ***\n"
 	cp app/config/parameters.yml.dist app/config/parameters.yml
 	sed -i "s/database_user:[ ]*root/database_user: $DB_USER/g" ./app/config/parameters.yml
 	sed -i "s/database_password:[ ]*~/database_password: $DB_PASSWORD/g" ./app/config/parameters.yml
 	sed -i "s/database_name:[ ]*[a-zA-Z0-9_]*/database_name: $DB_NAME/g" ./app/config/parameters.yml

 	# --- Composer Install ---

   	echo -e "\n*** Download and install composer ***\n"
 	wget -cq https://getcomposer.org/installer
 	php ./installer --install-dir=/usr/bin --filename=composer
 	rm ./installer

   	echo -e "\n*** Run 'composer install' command ***\n"
 	composer install --prefer-dist --no-dev

 	# --- Install Oro applicatioin ---

   	echo -e "\n*** Run 'oro:install' command ***\n"
   	php app/console oro:install --env=prod --application-url="http://$APP_HOST/" --organization-name="Oro Acme Inc" --user-name="$APP_USER" --user-email="admin@example.com" --user-firstname="Bob" --user-lastname="Dylan" --user-password="$APP_PASSWORD" --sample-data=$APP_LOAD_DEMO_DATA --timeout=600 --no-debug
   	
   	echo -e "\n*** Run 'oro:api:doc:cache:clear' command ***\n"
   	php app/console oro:api:doc:cache:clear --env=prod
 	
 	# --- Perform final cleaning ---

   	echo -e "\n*** Perform final cleaning ***\n"
 	rm ubuntu-xenial-16.04-cloudimg-console.log
 	
   	echo -e "\n*** Oro application installation finished ***\n"

	# --------------------- Final words ---------------------

   	echo -e "\n************* Congratulations! *************\n"

   	echo -e "\nInstallation finished successfully!\n"

   	echo -n "Application url: "
   	if [ "localhost" = $APP_HOST ]; then
   	    echo "http://$APP_HOST:8000/"
   	else
   	    echo "http://$APP_HOST/"
   	fi
   	echo "Admin login: $APP_USER"
   	echo "Admin password: $APP_PASSWORD"

   	echo -e "\nFor instructions\n\t- how to get to database from host machine,\n\t- how to use ssh in guest machine,\n\t- etc.\nplease see instructions in Vagrantfile.README.md file.\n"

  SHELL
end
