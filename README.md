# CANVAS_LMS

- Como root instalar :
	sudo apt-get install -y postgresql-9.5

adduser sysadmin
	user: sysadmin pass: 12

visudo
	sysadmin    ALL=(ALL:ALL) ALL

- Cambiar a sysadmin:
	su sysadmin

sudo -u postgres createuser canvas --no-createdb \
   --no-superuser --no-createrole --pwprompt

	pass for canvas user: admin123

sudo -u postgres createdb canvas_production --owner=canvas


- Getting the code:
cd
sudo apt-get install -y git-core
git clone https://github.com/instructure/canvas-lms.git canvas
cd canvas
git checkout stable


sudo mkdir -p /var/canvas
sudo chown -R sysadmin /var/canvas
cp -av . /var/canvas

cd /var/canvas

- Dependency Installation:

sudo apt-get install -y software-properties-common
sudo add-apt-repository ppa:brightbox/ruby-ng
sudo apt-get update

sudo apt-get install -y ruby2.4 ruby2.4-dev zlib1g-dev libxml2-dev \
                       libsqlite3-dev postgresql libpq-dev \
                       libxmlsec1-dev curl make g++


curl -sL https://deb.nodesource.com/setup_10.x | sudo -E bash -
sudo apt-get install -y nodejs


sudo -u postgres createuser $USER
sudo -u postgres psql -c "alter user $USER with superuser" postgres

- Ruby Gems:

sudo gem install bundler --version 1.13.7
bundle _1.13.7_ install --path vendor/bundle

- Yarn Installation:

curl -sS https://dl.yarnpkg.com/debian/pubkey.gpg | sudo apt-key add -
echo "deb https://dl.yarnpkg.com/debian/ stable main" | sudo tee /etc/apt/sources.list.d/yarn.list
sudo apt-get update && sudo apt-get install -y yarn=1.10.1-1
sudo apt-get install -y python
yarn install


# Canvas default configuration
for config in amazon_s3 database \
  delayed_jobs domain file_store outgoing_mail security external_migration; \
  do cp config/$config.yml.example config/$config.yml; done


---- ESTO DE AQUI NO LO COPIO ---
cp config/dynamic_settings.yml.example config/dynamic_settings.yml
nano config/dynamic_settings.yml


cp config/database.yml.example config/database.yml
nano config/database.yml
	username: canvas
	password: admin123


cp config/outgoing_mail.yml.example config/outgoing_mail.yml
nano config/outgoing_mail.yml
	production:
		address: "smtp.gmail.com"
		port: "587"
		user_name: "user@gmail.com"
		password: "password"
		domain: "testcanvas.com"
		outgoing_address: "user@gmail.com"


cp config/domain.yml.example config/domain.yml
nano config/domain.yml
	production:
		domain: "testcanvas.com"
		ssl: false


cp config/security.yml.example config/security.yml
nano config/security.yml
	encryption_key: 123456789012345678900


- Generate Assets:

cd /var/canvas
mkdir -p log tmp/pids public/assets app/stylesheets/brandable_css_brands
touch app/stylesheets/_brandable_variables_defaults_autogenerated.scss
touch Gemfile.lock
touch log/production.log


sudo adduser --disabled-password --gecos canvas canvasuser
sudo chown -R canvasuser config/environment.rb log tmp public/assets \
                              app/stylesheets/_brandable_variables_defaults_autogenerated.scss \
                              app/stylesheets/brandable_css_brands Gemfile.lock config.ru 


----- ACA SE DEMORA UN HUEVO EN ESTOS 2 --------------
yarn install

RAILS_ENV=production bundle exec rake canvas:compile_assets

sudo chown -R canvasuser public/dist/brandable_css



- DATaBSE POPULATION

RAILS_ENV=production bundle exec rake db:initial_setup

- Canvas login information:
	CANVAS_LMS_ADMIN_EMAIL = admin@testcanvas.com
	CANVAS_LMS_ADMIN_PASSWORD = admin123@@@
	CANVAS_LMS_ACCOUNT_NAME = COMPANY
	CANVAS_LMS_STATS_COLLECTION = 1


# Apache configuration

sudo apt-get install -y passenger libapache2-mod-passenger apache2
sudo a2enmod rewrite
sudo a2enmod passenger
sudo a2enmod ssl


- Configure Canvas with Apache: disable any Apache VirtualHosts you don't want running

sudo unlink /etc/apache2/sites-enabled/000-default.conf

sudo nano /etc/apache2/sites-available/canvas.conf

- ADD THIS TO: canvas.conf

PassengerDefaultUser canvasuser
<VirtualHost *:80>
  ServerName testcanvas.com
  ServerAlias canvasfiles.testcanvas.com
  ServerAdmin admin@testcanvas.com
  DocumentRoot /var/canvas/public
  #RewriteEngine On
  #RewriteCond %{HTTP:X-Forwarded-Proto} !=https
  #RewriteCond %{REQUEST_URI} !^/health_check
  #RewriteRule (.*) https://%{HTTP_HOST}%{REQUEST_URI} [L]  
  ErrorLog /var/log/apache2/canvas_errors.log
  LogLevel warn
  CustomLog /var/log/apache2/canvas_access.log combined
  SetEnv RAILS_ENV production
  <Directory /var/canvas/public>
    Options All
    AllowOverride All
    Require all granted
    Options -MultiViews
  </Directory>
</VirtualHost>


sudo a2ensite canvas
sudo apt-get install -y libapache2-mod-xsendfile
---sudo apachectl -M | sort


# Cache configuration
 - Redis:

sudo add-apt-repository ppa:chris-lea/redis-server
sudo apt-get update
sudo apt-get install -y redis-server


cd /var/canvas/
sudo cp config/cache_store.yml.example config/cache_store.yml
sudo nano config/cache_store.yml

test:
    cache_store: redis_store
development:
    cache_store: redis_store
production:
    cache_store: redis_store


sudo chown canvasuser config/cache_store.yml
sudo chmod 400 config/cache_store.yml


cd /var/canvas/
sudo cp config/redis.yml.example config/redis.yml
sudo nano config/redis.yml

production:
  servers:
    - redis://localhost


sudo chown canvasuser config/redis.yml
sudo chmod 400 config/redis.yml



## Automated jobs

sudo ln -s /var/canvas/script/canvas_init /etc/init.d/canvas_init
sudo update-rc.d canvas_init defaults
sudo /etc/init.d/canvas_init start






sudo /etc/init.d/apache2 restart


Go..............
admin@testcanvas.com
admin123@@@




