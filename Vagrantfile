# -*- mode: ruby -*-
# vi: set ft=ruby :

JENKINS_VERSION = '2.107.3'

Vagrant.configure("2") do |vagrant|
    vagrant.vm.define('master', primary: true) do |vagrant|
        vagrant.vm.box = "bento/centos-7.4"
        vagrant.vm.hostname = 'jenkins.vagrant.test'
        vagrant.vm.network "forwarded_port", guest: 8080, host: 18080

        vagrant.vm.provision "shell", inline: <<-SHELL
            set -e

            cd /vagrant

            if docker ps > /dev/null 2>&1 ; then
                echo "Docker is installed and running"
            else
                yum install -y docker
                systemctl start docker
                systemctl enable docker
            fi

            if netstat -plan | grep -q ':50000 ' ; then
                echo "Jenkins master is running"
            else
                mkdir -p /jenkins
                chmod 0777 /jenkins
                docker run -d --restart unless-stopped \
                    -p 8080:8080 -p 50000:50000 \
                    -v /jenkins:/var/jenkins_home:z \
                    --name goodold_jenkins \
                    jenkins/jenkins:#{JENKINS_VERSION}
            fi

            for n in {1..12} F ; do
                if [ $n == F ] ; then
                    echo "Timeout waiting for jenkins to start" >&2
                    exit 1
                elif [ -f /jenkins/secrets/initialAdminPassword ] ; then
                    echo "Initial admin password: "
                    cat /jenkins/secrets/initialAdminPassword
                    break
                else
                    echo "Waiting for jenkins to start..."
                    sleep 5
                fi
            done
        SHELL
    end
end
