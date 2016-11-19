# -*- mode: ruby -*-
# vi: set ft=ruby :

Vagrant.configure(2) do |config|
  config.vm.box = "bento/ubuntu-14.04"
  config.vm.hostname = "ruby.dev"
  config.ssh.shell = "bash -c 'BASH_ENV=/etc/profile exec bash'"

  ["vmware_fusion"].each do |p|
    config.vm.provider(p) do |v, override|
      v.memory = "512"
    end
  end

  #config.vm.synced_folder ".", "/vagrant", nfs: true, mount_options: ['nolock,vers=3,udp']
  config.vm.synced_folder ".", "/vagrant"

  config.vm.provision "shell", inline: $shell

  config.vm.network "forwarded_port", guest: 3000, host: 3000, auto_correct: true
  config.vm.network "forwarded_port", guest: 4000, host: 4000, auto_correct: true
end

$shell = <<-'CONTENTS'
export DEBIAN_FRONTEND=noninteractive
MARKER_FILE="/usr/local/etc/vagrant_provision_marker"
APT_PROXY="http://privatenetwork.host:3142"
RUBYGEMS_PROXY="http://privatenetwork.host:8081/repository/rubygems/"

# Only provision once
if [ -f "${MARKER_FILE}" ]; then
  exit 0
fi

# Halt on error
set -e

# Setup swap
fallocate -l 2G /swapfile
chmod 600 /swapfile
mkswap /swapfile
swapon /swapfile
echo '/swapfile none swap defaults 0 0' >> /etc/fstab

# Set locale information
update-locale LANG=en_US.UTF-8 LANGUAGE=en_US.UTF-8 LC_ALL=en_US.UTF-8

# Install apt proxy
echo 'Acquire::http { Proxy "${APT_PROXY}"; };' > /etc/apt/apt.conf.d/00proxy.conf

# Update apt
apt-get update -q

# Install basic dependencies
apt-get install -qy bsdtar build-essential ca-certificates curl \
  checkinstall cmake libssl-dev libreadline-dev \
  libxml2-dev libxslt-dev libyaml-dev make software-properties-common

# Install JS runtime
apt-get install -qy nodejs

# Install other tools
apt-get install -yq git htop tmux unzip

# Import postmodern's public key to verify chruby and ruby-install
curl -sL https://raw.github.com/postmodern/postmodern.github.io/master/postmodern.asc > /home/vagrant/postmodern.asc
gpg --import /home/vagrant/postmodern.asc
rm /home/vagrant/postmodern.asc

# Install ruby-install
curl -sL https://github.com/postmodern/ruby-install/archive/v0.6.0.zip > /home/vagrant/ruby-install-0.6.0.zip
unzip /home/vagrant/ruby-install-0.6.0.zip && cd /home/vagrant/ruby-install-0.6.0 && make install
ruby-install
rm /home/vagrant/ruby-install-0.6.0.zip
rm -rf /home/vagrant/ruby-install-0.6.0

# Install chruby
curl -sL https://github.com/postmodern/chruby/archive/v0.3.9.zip > /home/vagrant/chruby-0.3.9.zip
unzip /home/vagrant/chruby-0.3.9.zip -d /home/vagrant
cd /home/vagrant/chruby-0.3.9 && make install
rm /home/vagrant/chruby-0.3.9.zip
rm -rf /home/vagrant/chruby-0.3.9

# Install Ruby 2.3.1
ruby-install ruby-2.3.1

# Setup chruby
echo "source /usr/local/share/chruby/chruby.sh" > /home/vagrant/.bash_profile
echo "source /usr/local/share/chruby/auto.sh" >> /home/vagrant/.bash_profile

# Setup RubyGems mirror
su -l -c "gem sources --add ${RUBYGEMS_PROXY}" vagrant

# Install bundler
su -l -c "gem install bundler" vagrant

# Setup Bundler for RubyGems mirror
su -l -c "bundle config mirror.https://rubygems.org ${RUBYGEMS_PROXY}" vagrant

# Install phantomjs
apt-get -y install phantomjs

# Install chrome
wget -q -O - "https://dl-ssl.google.com/linux/linux_signing_key.pub" | sudo apt-key add -
echo 'deb [arch=amd64] http://dl.google.com/linux/chrome/deb/ stable main' > /etc/apt/sources.list.d/google-chrome.list
apt-get update
apt-get -y install google-chrome-stable

# Install chromedriver
apt-get -y install unzip
cd /tmp
wget "https://chromedriver.storage.googleapis.com/2.23/chromedriver_linux64.zip"
unzip chromedriver_linux64.zip
mv chromedriver /usr/local/bin

# Install virtual framebuffer
apt-get -y install xvfb

# Automatically move into the shared folder, but only add the command
# if it's not already there.
grep -q 'cd /vagrant' /home/vagrant/.bash_profile || echo 'cd /vagrant' >> /home/vagrant/.bash_profile

# Install databases
apt-get -y install libsqlite3-dev

# Touch the marker file so we don't do this again
touch ${MARKER_FILE}

echo "all set, rock on!"
CONTENTS
