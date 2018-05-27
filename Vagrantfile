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

            if java -version > /dev/null 2>&1 ; then
                echo "Java is installed"
            else
                yum install -y java-1.8.0-openjdk-headless
            fi

            if patch --version > /dev/null 2>&1 ; then
                echo "Patch is installed"
            else
                yum install -y patch
            fi

            if [ -f /usr/local/bin/jenkins-cli ] ; then
                echo "jenkins-cli wrapper is installed"
            else
                cp /vagrant/jenkins-cli /usr/local/bin
            fi
            PATH="$PATH:/usr/local/bin"

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

                for n in {1..12} F ; do
                    if [ $n == F ] ; then
                        echo "Timeout waiting for jenkins to start" >&2
                        exit 1
                    elif [ -f /jenkins/secrets/initialAdminPassword -a -f /jenkins/users/admin/config.xml ] ; then
                        sleep 5

                        # Bypass the initial admin password step. Don't do this on a production system,
                        # but on a boxed dev environment this extra step is just annoying.
                        systemctl stop docker
                        sed -i 's/<passwordHash>.*/<passwordHash><\\/passwordHash>/' /jenkins/users/admin/config.xml
                        patch /jenkins/users/admin/config.xml /vagrant/config.xml.patch
                        systemctl start docker
                        # These credentials must be in sync with the hash in config.xml.patch.
                        # The path must be consistent with the path in the jenkins-cli wrapper.
                        for d in ~ ~vagrant ; do
                            echo 'admin:admin' > $d/.jenkins-credentials
                            chown vagrant:vagrant $d/.jenkins-credentials
                            chmod 0400 $d/.jenkins-credentials
                        done
                        echo "Please log on to the web interface with user 'admin' and password 'admin'"
                        echo "to install functionality that is not shipped in the Jenkins image."
                        echo "The Jenkins CLI can be conveniently accessed through the jenkins-cli utility"
                        echo "(just a convenient wrapper around the original jar file)"

                        break
                    else
                        echo "Waiting for jenkins to start..."
                        sleep 5
                    fi
                done
            fi
        SHELL
    end
end
