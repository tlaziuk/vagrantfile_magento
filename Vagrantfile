# -*- mode: ruby -*-
# vi: set ft=ruby :

require 'date'

GUESTIP = "192.168.33.10"

LOCALIP = "localhost"

LOCALPORT = 8080

LOCALDOMAIN = "#{LOCALIP}:#{LOCALPORT}"

HOSTNAME = GUESTIP

PREFIX = ""

KEY = LOCALDOMAIN

LOCALXML = <<XML
<?xml version="1.0"?>
<config>
    <global>
        <install>
            <date>#{Time.now.to_datetime.rfc3339}</date>
        </install>
        <crypt>
            <key>#{KEY}</key>
        </crypt>
        <disable_local_modules>false</disable_local_modules>
        <resources>
            <db>
                <table_prefix>#{PREFIX}</table_prefix>
            </db>
            <default_setup>
                <connection>
                    <host>localhost</host>
                    <username>root</username>
                    <password>root</password>
                    <dbname>scotchbox</dbname>
                    <initStatements>SET NAMES utf8</initStatements>
                    <model>mysql4</model>
                    <type>pdo_mysql</type>
                    <pdoType>1</pdoType>
                    <active>1</active>
                </connection>
            </default_setup>
        </resources>
        <session_save>files</session_save>
    </global>
    <admin>
        <routers>
            <adminhtml>
                <args>
                    <frontName>admin</frontName>
                </args>
            </adminhtml>
        </routers>
    </admin>
</config>
XML

SH = <<SCRIPT
wget -q -O - "https://raw.githubusercontent.com/Chikoumi/Ioncube-Autoinstall/master/eng_ioncube.sh" | sudo bash
cat > /var/www/public/app/etc/local.xml <<- EOF
#{LOCALXML}
EOF
wget -q -O /usr/local/bin/n98-magerun.phar https://files.magerun.net/n98-magerun.phar
chmod a+x /usr/local/bin/n98-magerun.phar
ln -f -s /var/www/public /home/vagrant/www
sudo apt-get install pv htop
cd /home/vagrant/www
for FILE in *.sql
do
    if [ -s "$FILE" ]; then
        n98-magerun.phar db:import "$FILE"
    fi
done
for FILE in *.sql.gz
do
    if [ -s "$FILE" ]; then
        n98-magerun.phar db:import "$FILE" --compression=gz
    fi
done
n98-magerun.phar db:query "UPDATE #{PREFIX}core_config_data SET value = 'http://#{LOCALDOMAIN}/' WHERE path IN ('web/unsecure/base_url', 'web/secure/base_url');"
n98-magerun.phar db:query "UPDATE #{PREFIX}core_config_data SET value = '0' WHERE path IN ('dev/css/merge_css_files', 'dev/js/merge_files');"
n98-magerun.phar db:query "DELETE FROM #{PREFIX}core_config_data WHERE (path LIKE 'web/unsecure/%' OR path LIKE 'web/secure/%') AND scope_id != 0;"
n98-magerun.phar cache:disable
n98-magerun.phar cache:clean
n98-magerun.phar cache:flush
n98-magerun.phar cache:dir:flush
n98-magerun.phar index:reindex:all
SCRIPT

Vagrant.configure("2") do |config|
    config.vm.box = "scotch/box"
    config.vm.box_version = "~> 2.5"
    config.vm.network "private_network", ip: GUESTIP
    config.vm.network "forwarded_port", guest: 80, host: LOCALPORT
    config.vm.hostname = HOSTNAME
    config.vm.synced_folder ".", "/var/www/public", :mount_options => ["dmode=777", "fmode=666"]
    config.vm.provision "shell", inline: SH
end
