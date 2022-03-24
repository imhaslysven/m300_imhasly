# LB2 - Gitlab-Installation automatisieren

## Inhaltsverzeichnis

Inhaltsverzeichnis für LB2.md von Sven Imhasly.

- [Einleitung](#Einleitung)
- [Service](#Service)
  - [Übersicht](#Übersicht)
- [Code](#Code)
	- [Starten](#Start)
- [Testing](#Testing)
- [Quellenverzeichnis](#Quellenverzeichnis)

## Einleitung
Das Ziel der LB2 ist, ein Dienst mithilfe von Vagrant zu automatisieren. 
Für diese Aufgabe habe ich mich mit Ricardo Frei zusammengeschlossen. Wir haben uns dazu entschieden den Service `Gitlab` anzuschauen.

Mein Ergebniss ist im Vagrantfile abrufbar. Mithilfe den einzelnen Commits ist ersichtlich wann und was von mir erledigt wurde.

## Service
Unser eigen gehostetes Gitlab soll sich selbstständig installieren und auf dem neusten Stand verfügbar sein.

Das Ziel ist es, dass der Service unter folgender URL [http://localhost:8080](http://localhost:8080) erreichbar ist. 

Dabei kann man sich mit dem Standarduser root anmelden können. Nicht verwundern, zum Beginn muss das Passwort geändert werden.

Username | Password
---------|-----------
root     | 5iveL!fe 


### Übersicht
![Übersicht Service](https://github.com/imhaslysven/m300_imhasly/blob/main/Umgebung_m300.PNG)

## Code

<pre><code>
VAGRANTFILE_API_VERSION = "2"

Vagrant.configure(VAGRANTFILE_API_VERSION) do |config|
  config.vm.box = "ubuntu/bionic64"
  config.vm.hostname = "gitlab.imhasly"

  if Vagrant.has_plugin?("vagrant-vbguest")
    config.vbguest.auto_update = false
  end

  config.vm.network "forwarded_port", guest: 80, host: 8080
  config.vm.network "forwarded_port", guest: 22, host: 8022

  config.vm.network "private_network", ip: "192.168.0.10"
  config.vm.synced_folder ".", "/vagrant", type: "rsync", rsync__exclude: [".git/"]

  config.vm.provider "virtualbox" do |vb|
    vb.name = "Gitlab Imhasly"
    vb.memory = "4096"
  end
</code></pre>

Hier wurde die Basic Vagrant Config für die VM gemacht. Zu dieser gehört Hostname, Guest Addition von VirtualBox deaktivieren, Portforwarding, Networking, Syncs und die Konfig selbst.

<pre><code>
 config.vm.provision "shell", inline: <<-SHELL
    sudo apt-get update
    sudo apt-get install -y curl openssh-server ca-certificates
    debconf-set-selections <<< "postfix postfix/mailname string $HOSTNAME"
    debconf-set-selections <<< "postfix postfix/main_mailer_type string 'Internet Site'"
    DEBIAN_FRONTEND=noninteractive sudo apt-get install -y postfix
    if [ ! -e /vagrant/ubuntu-bionic-gitlab-ce_12.9.3-ce.0_amd64.deb ]; then
        wget --content-disposition -O /vagrant/ubuntu-bionic-gitlab-ce_12.9.3-ce.0_amd64.deb https://packages.gitlab.com/gitlab/gitlab-ce/packages/ubuntu/bionic/gitlab-ce_12.9.3-ce.0_amd64.deb/download.deb
    fi
    sudo dpkg -i /vagrant/ubuntu-bionic-gitlab-ce_12.9.3-ce.0_amd64.deb
    sudo gitlab-ctl reconfigure
  SHELL
end

Hier wurden dann die ersten Commands auf der Maschine in der Shell ausgeführt. Neusten Packete und die einzelne Dienste installiert. Danach folgt schlussendlich die endgültige Git-Konfiguration.

</code></pre>


### Start
1. Herunterladen der Dateien und in dem Verzeichnis, welchem das `Vagranfile` liegt Punkt 2. ausführen.
2. `vagrant up`
3. Auf die Website: [http://localhost:8080](http://localhost:8080) verbinden.
4. Passwort neu setzten. (Muss 8 Zeichen lang sein!)
5. Anmelden mit Username. `root` und zuvor gesetztem Passwort. 
6. Feel-Free Projekte usw. zu erstellen. 

### Testing
> Die Installation mit `vagrant up` hat erfolgreich geklappt. Jetzt kann das Ganze noch mit dem Befehl `vagrant global-status` geprüft werden. Hier sollte die laufende Maschine ersichtlich sein. Über ihre ID könnte sie in einem nächsten Schritt auch gelöscht werden.



## Quellenverzeichnis

[How to Install Debian using Vagrant](https://www.regur.net/blog/how-to-install-debian-using-vagrant/)</br>
[Basic Vagrant Gitlab Server](https://gist.github.com/cjtallman/b526d8c7d8b910ba4fd41eb51cd5405b)</br>
[Deploy Gitlab via Vagrant](https://www.exoscale.com/syslog/deploy-gitlab-on-ubuntu-12-04-with-vagrant/)</br>