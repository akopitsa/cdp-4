$hostnamepuppetserver = "puppettestserver.loc"
$ipaddresspuppetserver = "192.168.99.101"
$hostnamepuppetagent = "puppettestagent.loc"
$ipaddresspuppetagent = "192.168.99.102"
Vagrant.configure("2") do |config|
    config.vm.box = "centos/7"
    config.vm.define "puppetserver" do |puppetserver|
    puppetserver.vm.hostname = $hostnamepuppetserver
    puppetserver.vm.network "private_network", ip: $ipaddresspuppetserver
    puppetserver.vm.provider :virtualbox do |server|
      server.name = "puppettestserver"
      server.memory = 3096
      server.cpus = 2
    end
    puppetserver.vm.provision "shell", inline: <<-SHELL
      yum update
      yum install -y ntp vim mc git tree net-tools
      timedatectl set-timezone Europe/Kiev
      ntpdate pool.ntp.org
      sed -i 's/server 0.centos.pool.ntp.org/server 0.ua.pool.ntp.org/g' /etc/ntp.conf
      sed -i 's/server 1.centos.pool.ntp.org/server 1.ua.pool.ntp.org/g' /etc/ntp.conf
      sed -i 's/server 2.centos.pool.ntp.org/server 2.ua.pool.ntp.org/g' /etc/ntp.conf
      sed -i 's/server 3.centos.pool.ntp.org/server 3.ua.pool.ntp.org/g' /etc/ntp.conf
      systemctl restart ntpd
      systemctl enable ntpd
      rpm -ivh https://yum.puppetlabs.com/puppetlabs-release-pc1-el-7.noarch.rpm
      yum -y install http://dl.fedoraproject.org/pub/epel/epel-release-latest-7.noarch.rpm
      yum -y install https://yum.theforeman.org/releases/1.16/el7/x86_64/foreman-release.rpm
      yum -y install puppetserver
      yum -y install foreman-installer
#      sed -i 's/-Xms2g -Xmx2g/-Xms3g -Xmx3g/g' /etc/sysconfig/puppetserver
    if [ -f '/etc/puppetlabs/puppet/autosign.conf' ]; then
      echo "Autosign file already exist"
    else
    echo "*.loc" >> /etc/puppetlabs/puppet/autosign.conf
    fi
    if  grep -Fxq "[main]" /etc/puppetlabs/puppet/puppet.conf
    then
    echo "Puppet Server Settings Already addded"
    else
    echo """
dns_alt_names = #{$hostnamepuppetserver},server
autosign = /etc/puppetlabs/puppet/autosign.conf
[main]
certname = #{$hostnamepuppetserver}
server = #{$hostnamepuppetserver}
environment = production
runinterval = 2m
""" >> /etc/puppetlabs/puppet/puppet.conf
fi
echo """:backends:
  - yaml
  - eyaml
:hierarchy:
  - "nodes/%{::trusted.certname}"
  - "nodes/%{::fqdn}"
  - common

:yaml:
# datadir is empty here, so hiera uses its defaults:
# - /etc/puppetlabs/code/environments/%{environment}/hieradata on *nix
# When specifying a datadir, make sure the directory exists.
  :datadir: '/etc/puppetlabs/code/environments/%{::environment}/hieradata'
""" > /etc/puppetlabs/puppet/hiera.yaml
/opt/puppetlabs/puppet/bin/gem install hiera-eyaml
/opt/puppetlabs/puppet/bin/eyaml createkeys
systemctl start puppetserver
systemctl enable puppetserver
#foreman-installer
/opt/puppetlabs/puppet/bin/gem install r10k
if  [ -f "/etc/puppetlabs/puppet/r10k.yaml" ];
then
echo "R10K Settings Already addded"
else
echo ''':cachedir: '/var/cache/r10k'
:sources:
  cdp:
    remote: 'https://github.com/akopitsa/cdp-control-repo.git'
    basedir: '/etc/puppetlabs/code/environments'
    prefix: false
''' >> /etc/puppetlabs/puppet/r10k.yaml
fi
/opt/puppetlabs/puppet/bin/r10k deploy environment production -pv -c /etc/puppetlabs/puppet/r10k.yaml
/opt/puppetlabs/puppet/bin/r10k deploy environment development -pv -c /etc/puppetlabs/puppet/r10k.yaml
cat <<- EOF > cron.jobs
*/5 * * * *  /opt/puppetlabs/puppet/bin/r10k deploy environment production -pv -c /etc/puppetlabs/puppet/r10k.yaml
*/5 * * * *  /opt/puppetlabs/puppet/bin/r10k deploy environment development -pv -c /etc/puppetlabs/puppet/r10k.yaml
EOF
crontab cron.jobs
SHELL
 end
  config.vm.define "puppetagent" do |puppetagent|
    puppetagent.vm.hostname = $hostnamepuppetagent
    puppetagent.vm.network "private_network", ip: $ipaddresspuppetagent
    puppetagent.vm.provider :virtualbox do |agent|
      agent.name = "PuppetAgent1"
      agent.memory = 1024
      agent.cpus = 2
  end
    puppetagent.vm.provision "shell", inline: <<-SHELL
      yum update
      yum install -y vim ntpdate mc tree net-tools
      timedatectl set-timezone Europe/Kiev
      ntpdate pool.ntp.org
      rpm -ivh https://yum.puppetlabs.com/puppetlabs-release-pc1-el-7.noarch.rpm
      yum -y install puppet-agent
      echo "#{$ipaddresspuppetserver} #{$hostnamepuppetserver}" >> /etc/hosts
      if  grep -Fxq "[main]" /etc/puppetlabs/puppet/puppet.conf
      then
      echo "Settings Already addded"
      else
      echo """
[main]
certname = #{puppetagent.vm.hostname}
server = #{$hostnamepuppetserver}
environment = production
runinterval = 1m
""" >> /etc/puppetlabs/puppet/puppet.conf
      fi
      /opt/puppetlabs/bin/puppet resource service puppet ensure=running enable=true
    SHELL
    end
end
# Initial credentials are admin / HFaLJXUrCFmpdqbB