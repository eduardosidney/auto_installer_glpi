# GLPI installer for Ubuntu and Debian servers

# Download the file and give the permission to run

```
wget https://raw.githubusercontent.com/eduardosidney/auto_installer_glpi/main/install.sh
sudo chmod +x install.sh

sudo ./install.sh
 
````
#!/bin/bash

####### Variables #######
Setup_Variables()
{
        DBPASSWORD=$(whiptail --passwordbox "set database password")
}
function deb_based_install()
{
    echo -e "\033[1;30m****** Installing dependences ******* \033[0m"
    apt install -y xz-utils bzip2 unzip curl

    sleep 10
    echo -e "\033[1;30m****** Installing PHP Modules ******* \033[0m"
    apt install -y apache2 libapache2-mod-php php-soap php-cas php php-{apcu,cli,common,curl,gd,imap,ldap,mysql,xmlrpc,xml,mbstring,bcmath,intl,zip,bz2}

    sleep 10
    echo -e "\033[1;30m****** Downloading GLPI on /var/www/html  ******* \033[0m"
    wget -O- https://github.com/glpi-project/glpi/releases/download/10.0.2/glpi-10.0.2.tgz | tar -zx -C /var/www/html/

    sleep 10
    echo -e "\033[1;30m****** Changing Permissions ******* \033[0m"
    chown www-data:www-data /var/www/html/glpi -Rf
    chmod -R 755 /var/www/html
    sleep 10

    echo -e "\033[1;30m****** Installing MariaDB-Server ******* \033[0m"
    apt install -y mariadb-server
}
function mysql_sec_install()
{
        echo -e "\033[1;30m****** Configuration MariaDB \033[0m"
        mysql_secure_installation <<EOF
    y
    ${DBPASSWORD}
    ${DBPASSWORD}
    y
    y
    y
    y
EOF
}
function database_install()
{
        echo -e "\033[1;30m****** Configuring Database ******* \033[0m"
        mysql -e "CREATE DATABASE glpi CHARACTER SET utf8"
        mysql -e "CREATE USER 'glpi'@'localhost' IDENTIFIED BY 'Glpi#1'"
        mysql -e "GRANT ALL PRIVILEGES ON glpi.* to 'glpi'@'localhost' WITH GRANT OPTION";
        mysql_tzinfo_to_sql /usr/share/zoneinfo | mysql -p -u root mysql
        mysql -e "GRANT SELECT ON mysql.time_zone_name TO 'glpi'@'localhost';"
        mysql -e "FLUSH PRIVILEGES;"
        echo -e "\033[1;30m****** Installing ******* \033[0m"
        php /var/www/html/glpi/bin/console glpi:database:install --db-host=localhost --db-name=glpi --db-user=glpi --db-password=Glpi#1
}
function permissions_and_enable()
{
        echo -e "\033[1;30m****** More Permissions ******* \033[0m"
        chown www-data:www-data /var/www/html/glpi/files -Rf
        echo -e "\033[1;30m****** Enable and start Apache ******* \033[0m"
        systemctl enable apache2
        systemctl restart apache2
}
function main()
{
    deb_based_install -e "error"
    Setup_Variables -e "error"
    database_install -e "error"
    permissions_and_enable -e "error"
}
