$install_confluence = <<SCRIPT
wget https://www.atlassian.com/software/confluence/downloads/binary/atlassian-confluence-6.2.1-x64.bin -O /vagrant/
#------------------------------------------------------
echo "rmiPort$Long=8000
app.install.service$Boolean=yes
existingInstallationDir=/usr/local/Confluence
sys.confirmedUpdateInstallationString=false
sys.languageId=en
sys.installationDir=/opt/atlassian/confluence
app.confHome=/var/atlassian/application-data/confluence
executeLauncherAction$Boolean=false
httpPort$Long=8090
portChoice=default" > /tmp/response.varfile
#-------------------------------------------------------
chmod a+x /vagrant/atlassian-confluence-6.2.1-x64.bin
/vagrant/atlassian-confluence-6.2.1-x64.bin -q -varfile /tmp/response.varfile
SCRIPT

$install_mysql = <<SCRIPT
apt-get update && apt-get upgrade -y
echo "mysql-server-5.7 mysql-server/root_password password root" | debconf-set-selections
echo "mysql-server-5.7 mysql-server/root_password_again password root" | debconf-set-selections
apt-get -y install mysql-server-5.7
wget https://cdn.mysql.com//Downloads/Connector-J/mysql-connector-java-5.1.42.tar.gz
tar -zxvf mysql-connector-java-5.1.42.tar.gz
cp mysql-connector-java-5.1.42/mysql-connector-java-5.1.42-bin.jar /opt/atlassian/confluence/confluence/WEB-INF/lib/
rm -rf mysql-connector-java*
mysql -u root -proot -e "CREATE DATABASE confluence CHARACTER SET utf8 COLLATE utf8_bin;
GRANT ALL PRIVILEGES ON confluence.* TO 'confluenceuser'@'localhost' IDENTIFIED BY 'confluencepass';"
/opt/atlassian/confluence/bin/shutdown.sh
/opt/atlassian/confluence/bin/startup.sh
SCRIPT

$install_nginx = <<SCRIPT
apt-get install -y nginx
mkdir -p /etc/nginx/ssl
openssl req -new -x509 -nodes -days 365 -newkey rsa:2048 -keyout /etc/nginx/ssl/nginx.key -out /etc/nginx/ssl/nginx.cert -subj /CN=*.mik.com
echo "server {
    listen 80;
    return 301 https://\$host\$request_uri;
}

server {
    listen 443;

    ssl on;
    ssl_certificate           /etc/nginx/ssl/nginx.cert;
    ssl_certificate_key       /etc/nginx/ssl/nginx.key;
    ssl_protocols TLSv1 TLSv1.1 TLSv1.2;
    ssl_prefer_server_ciphers on;
    ssl_ciphers HIGH:!aNULL:!eNULL:!EXPORT:!CAMELLIA:!DES:!MD5:!PSK:!RC4;
    ssl_session_timeout 1d;
    ssl_session_cache  builtin:1000  shared:SSL:10m;
    ssl_stapling on;
    ssl_stapling_verify on;
    add_header Strict-Transport-Security max-age=15768000;
    client_max_body_size 250M;

    location / {
      proxy_pass             http://localhost:8090;
    }
}" > /etc/nginx/sites-available/default
service nginx reload
SCRIPT

$ssh_key = <<SCRIPT
echo "-----BEGIN RSA PRIVATE KEY-----
here body of key
-----END RSA PRIVATE KEY-----" > /root/.ssh/id_rsa
chmod 400 /root/.ssh/id_rsa
ssh -i /root/.ssh/id_rsa -o ConnectTimeout=10 -o StrictHostKeyChecking=no ubuntu@your_remote_ip "sudo cp .ssh/authorized_keys /root/.ssh/; grep 'GatewayPorts yes' /etc/ssh/sshd_config || sudo bash -c 'echo GatewayPorts yes >> /etc/ssh/sshd_config && sudo service ssh restart'"
SCRIPT

$ssh_port = <<SCRIPT
ssh -o StrictHostKeyChecking=no -fNR 0.0.0.0:443:localhost:443 root@your_remote_ip
ssh -o StrictHostKeyChecking=no -fNR 0.0.0.0:80:localhost:80 root@your_remote_ip
SCRIPT

Vagrant.configure("2") do |config|
	config.vm.box = "ubuntu/xenial64"
	config.vm.provider "virtualbox" do |vb|
		vb.memory = "2048"
	end
	config.vm.synced_folder ".", "/vagrant/"
	config.vm.provision "shell", inline: $install_confluence
	config.vm.provision "shell", inline: $install_mysql
	config.vm.provision "shell", inline: $install_nginx
	config.vm.network "private_network", ip: "192.168.33.10"
	config.vm.provision "shell", inline: $ssh_key
	config.vm.provision "shell", inline: $ssh_port, run: "always"
end