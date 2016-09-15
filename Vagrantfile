# -*- mode: ruby -*-
# vi: set ft=ruby :

Vagrant.configure(2) do |config|
  config.vm.box = "geerlingguy/ubuntu1604"

  config.vm.provider "virtualbox" do |vb|
    vb.memory = "2048"
    vb.name = "mysql-to-postgres"
  end

  config.vm.provision "shell", inline: <<-SHELL
    set -e
    MY_PASSWD=some_password
    # intall things ###########################################
    sudo apt-get update
    echo "mysql-server mysql-server/root_password password $MY_PASSWD" | debconf-set-selections
    echo "mysql-server mysql-server/root_password_again password $MY_PASSWD" | debconf-set-selections
    sudo apt-get install postgresql-9.5 mysql-server postgresql-9.5-mysql-fdw libmysqlclient-dev -y

    # mysql ###################################################
    echo 'create database test;' | mysql -u root --password=$MY_PASSWD
    echo 'create table table1 (id int, txt text);' | mysql -u root --password=$MY_PASSWD test
    echo "insert into table1 (id, txt) values (1, 'row1'), (2, 'row2');" | mysql -u root --password=$MY_PASSWD test

    # postgresql ##############################################
    sudo -u postgres createdb test
    sudo -u postgres psql test -c "create table table1 (id serial not null primary key, txt_other_name text);"
    sudo -u postgres psql test -c "create extension mysql_fdw;"
    sudo -u postgres psql test -c "CREATE SERVER mysql_server FOREIGN DATA WRAPPER mysql_fdw OPTIONS (host '127.0.0.1', port '3306');"
    sudo -u postgres psql test -c "CREATE USER MAPPING FOR postgres SERVER mysql_server OPTIONS (username 'root', password '$MY_PASSWD');"
    sudo -u postgres psql test -c "CREATE FOREIGN TABLE mysql_table1(id int, txt text) SERVER mysql_server OPTIONS (dbname 'test', table_name 'table1');"
    sudo -u postgres psql test -c "select * from table1;"
    sudo -u postgres psql test -c "insert into table1 (id, txt_other_name) select id, txt from mysql_table1;"
    # if it shows two records it means that you imported the mysql table into your postgresql table
    sudo -u postgres psql test -c "select * from table1;"
  SHELL
end
