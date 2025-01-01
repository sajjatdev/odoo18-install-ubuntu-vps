# Update the package list to ensure you are installing the latest versions of the packages.
sudo apt-get update

# Upgrade installed packages to their latest versions.
sudo apt-get upgrade

# Install Python 3 pip and other essential Python development libraries.
sudo apt-get install -y python3-pip
sudo apt-get install -y python3-dev libxml2-dev libxslt1-dev zlib1g-dev libsasl2-dev libldap2-dev build-essential libssl-dev libffi-dev libmysqlclient-dev libjpeg-dev libpq-dev libjpeg8-dev liblcms2-dev libblas-dev libatlas-base-dev

# Create a symbolic link for Node.js and install Less and Less plugins.
sudo ln -s /usr/bin/nodejs /usr/bin/node
sudo npm install -g less less-plugin-clean-css
sudo apt-get install -y node-less

# Install PostgreSQL (the database used by Odoo) and create a new user for Odoo 18.
sudo apt-get install -y postgresql
sudo su - postgres
createuser --createdb --username postgres --no-createrole --superuser --pwprompt odoo18
Exit

# Create a system user for Odoo 18 and install Git to clone the Odoo source code.
sudo adduser --system --home=/opt/odoo18 --group odoo18
sudo apt-get install -y git
sudo su - odoo18 -s /bin/bash
git clone https://www.github.com/odoo/odoo --depth 1 --branch master --single-branch .
Exit

# Install Python virtual environment and set up the Odoo environment.
sudo apt install -y python3-venv
sudo python3 -m venv /opt/odoo18/venv

# Switch to root, navigate to the Odoo directory, activate the virtual environment, and install required Python packages.
sudo -s
cd /opt/odoo18/
source venv/bin/activate
pip install -r requirements.txt

# Install wkhtmltopdf (used for printing PDF reports in Odoo) and resolve any missing dependencies.
sudo wget https://github.com/wkhtmltopdf/wkhtmltopdf/releases/download/0.12.5/wkhtmltox_0.12.5-1.bionic_amd64.deb
wget http://archive.ubuntu.com/ubuntu/pool/main/o/openssl/libssl1.1_1.1.0g-2ubuntu4_amd64.deb

sudo dpkg -i libssl1.1_1.1.1f-1ubuntu2_amd64.deb
sudo apt-get install -y xfonts-75dpi
sudo dpkg -i wkhtmltox_0.12.5-1.bionic_amd64.deb
sudo apt install -f
deactivate

# Configure the Odoo instance by copying the default config file and editing it to suit your needs.
sudo cp /opt/odoo18/debian/odoo.conf /etc/odoo18.conf
sudo nano /etc/odoo18.conf

# Odoo configuration file settings, including database connection and log file location.
# [options]
# ; This is the password that allows database operations:
# ; admin_passwd = admin
# db_host = localhost
# db_port = 5432
# db_user = odoo18
# db_password = 123456
# addons_path = /opt/odoo18/addons
# default_productivity_apps = True
# logfile = /var/log/odoo/odoo18.log

# Set correct permissions on the Odoo configuration file to ensure security.
sudo chown odoo18: /etc/odoo18.conf
sudo chmod 640 /etc/odoo18.conf

# Create a directory for Odoo log files and set appropriate ownership.
sudo mkdir /var/log/odoo
sudo chown odoo18:root /var/log/odoo

# Create a systemd service file for Odoo 18 to manage it as a service.
sudo nano /etc/systemd/system/odoo18.service

# Odoo systemd service configuration.
# [Unit]
# Description=Odoo18
# Documentation=http://www.odoo.com
# [Service]
# Ubuntu/Debian convention:
# Type=simple
# User=odoo18
# ExecStart=/opt/odoo18/venv/bin/python3.12 /opt/odoo18/odoo-bin -c /etc/odoo18.conf
# [Install]
# WantedBy=default.target

# Set permissions and ownership on the systemd service file.
sudo chmod 755 /etc/systemd/system/odoo18.service
sudo chown root: /etc/systemd/system/odoo18.service
