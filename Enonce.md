
# Partie 1 : Administration avec Portainer

  - Installez l'édition communautaire de **Portainer** à l'aide de la documentation officielle : https://docs.portainer.io/start/install/server/docker/linux

  - Sous **VScode**, Installer le plugin **Docker** et visualisez les objets docker

  
# Partie 2 : Harbor + Trivy
Il est question d'installer **harbor** et de configurer le scanner **trivy**.

 > :warning: ***Vous pouvez travailler en tant qu'utilisateur root directement***

  
### 1 - Téléchargement et extraction de l'archive

	wget https://github.com/goharbor/harbor/releases/download/v2.0.0/harbor-online-installer-v2.0.0.tgz
	tar xvf harbor-online-installer-v2.0.0.tgz


### 2 - Génération des certificats

#### a - Clé privée du CA :

	openssl genrsa -out ca.key 4096

#### b - Certificat du CA
	openssl req -x509 -new -nodes -sha512 -days 3650 \
	-subj "/C=CN/ST=Colombo/L=Colombo/O=Organization/OU=Personal/CN=harbor-registry.com" \
	-key ca.key \
	-out ca.crt

#### c - Certificat du serveur
	openssl genrsa -out harbor-registry.com.key 4096
 
#### d - Génération de la CSR
	openssl req -sha512 -new \
	-subj "/C=CN/ST=Colombo/L=Colombo/O=Organization/OU=Personal/CN=harbor-registry.com" \
	-key harbor-registry.com.key \
	-out harbor-registry.com.csr

#### e - Génération d'un fichier d'extension x509 v3

	cat > v3.ext <<-EOF
	authorityKeyIdentifier=keyid,issuer
	basicConstraints=CA:FALSE
	keyUsage = digitalSignature, nonRepudiation, keyEncipherment, dataEncipherment
	extendedKeyUsage = serverAuth
	subjectAltName = @alt_names
	[alt_names]
	DNS.1=harbor-registry.com
	DNS.2=harbor-registry
	DNS.3=host-name
	EOF

  
#### f - Génération du certificat

	openssl x509 -req -sha512 -days 3650 \
	-extfile v3.ext \
	-CA ca.crt -CAkey ca.key -CAcreateserial \
	-in harbor-registry.com.csr \
	-out harbor-registry.com.crt


#### g - Dépôt du certificat pour harbort

	mkdir -p /data/cert/
	cp harbor-registry.com.crt /data/cert/
	cp harbor-registry.com.key /data/cert/
  

#### h - configuration des certificats pour docker

	mkdir -p /etc/docker/certs.d/harbor-registry.com/
	openssl x509 -inform PEM -in harbor-registry.com.crt -out harbor-registry.com.cert
	cp harbor-registry.com.cert /etc/docker/certs.d/harbor-registry.com/
	cp harbor-registry.com.key /etc/docker/certs.d/harbor-registry.com/
	cp ca.crt /etc/docker/certs.d/harbor-registry.com/
  

#### i - Redémarrage de docker
	systemctl restart docker

  

### 3 - Déploiement de harbor

#### a - Fichier de configuration de harbor
Dans le dossier **harbor** téléchargé, il faut créer un fichier de configuration à partir du fichier **template harbor.yml.tmpl**

	cp harbor.yml.tmpl harbor.yml
Dans ce fichier, il faudra changer les lignes suivantes comme suit :
	
	hostname: harbor-registry.com
	# https related config
	https:
	  # https port for harbor, default is 443
	  port: 443
	  # The path of cert and key files for nginx
	  certificate: /data/cert/harbor-registry.com.crt
	  private_key: /data/cert/harbor-registry.com.key

#### b -Lancer l'exécutable prepare
	./prepare
 

#### c - Lancement du docker compose.
Un fichier docker-compose a été créé, il faudrait le lancer
	
	docker-compose up -d

  

### 4 - Accès à Harbor

#### a - Autorisez le registre dans docker (registres insécures)

	mkdir -p /etc/docker/
	cat > /etc/docker/daemon.json <<-EOF
	{
	“insecure-registries”: [
	“harbor-registry.com”
	]
	}
	EOF

  

#### b - Connexion en CLI et push/pull d'image
	
	docker login harbor-registry.com
	docker pull httpd
	docker tag httpd:latest harbor-registry.com/test/httpd:latest

  

#### c - Connexion à l'IHM 
Connectez vous sur l'IHM en ssl. Il faudrait peutêtre modifier votre ***/etc/host***
**url** : **https://harbor-registry.com**

  

### 5 - Configuration de Trivy 
Il faut se déplacer dans le dossier harbor d'insallation.

#### a - Création des variables d'environnement

	mkdir -p ./common/config/trivy-adapter
	cat << EOF > ./common/config/trivy-adapter/env
	SCANNER_LOG_LEVEL=trace
	SCANNER_STORE_REDIS_URL=redis://redis:6379
	SCANNER_JOB_QUEUE_REDIS_URL=redis://redis:6379
	SCANNER_TRIVY_CACHE_DIR=/home/scanner/.cache/trivy
	SCANNER_TRIVY_REPORTS_DIR=/home/scanner/.cache/reports
	SCANNER_TRIVY_VULN_TYPE=os,library
	SCANNER_TRIVY_SEVERITY=UNKNOWN,LOW,MEDIUM,HIGH,CRITICAL
	SCANNER_TRIVY_IGNORE_UNFIXED=false
	SCANNER_TRIVY_DEBUG_MODE=false
	EOF

#### b - Création et lancement du docker-compose

Dans les sources, récupérer le fichier **docker-compose.override.yml** et le déposer dans le dossier harbor. Créer aussi les réperoires suivants : */home/ec2-user/voting-app/harbor/data/trivy-adapter/trivy* et */home/ec2-user/voting-app/harbor/data/trivy-adapter/reports*
		
	
	mkdir -p /home/ec2-user/voting-app/harbor/data/trivy-adapter/trivy /home/ec2-user/voting-app/harbor/data/trivy-adapter/reports

Enfin, Lancez le docker compose

	docker-compose up -d
  
#### c - Ajout du scanner dans l'IHM harbor 

Il faut à présent terminer la configuration dans l'IHM harbor en rajoutant un nouveau scanner. Il faut aller dans **Interrogation Services > New Scanner**

- *Name* : **my-trvy-scanner**
- *Endpoint* : **http://trivy-adapter:8080**

  
# Partie 3 : Grafana + Prometheus

### 1 - Déployer la stack prometheus/Grafana sur docker.

Un code source est fournit avec l'énoncé, dans le dossier **monitoring**. Là dedans, se trouve un docker-compose et des fichiers de configuration.

	git clone https://github.com/ulrichmonji/projet-ajc.git 
	cd projet-ajc/monitoring/
    docker-compose up -d 
  

### 2 - Configuration de l'export des métriques
Une fois la stack déployée, rajouter l'export de métrique dans la configuration de  docker. Il faudrait rajouter les options suivantes  : 

		"metrics-addr" : "<YOUR IP>:9323",
		"experimental" : true
***Attention à ne pas écraser une configuration qui serait déja existante...***
Ne pas oublier de redémarrer docker

	cat > /etc/docker/daemon.json <<-EOF
	{
		"metrics-addr" : "<YOUR IP>:9323",
		"experimental" : true
	}
	EOF
	sudo systemctl restart docker

  Des métriques devraient être disponibles à cette url : http://<YOUR IP:9393/metrics>

  

### 3 - Prise en compte dans Prometheus 
Prendre en compte la nouvelle métrique avec dans Prometheus. Pour celà Il faut modifier le fichier **prometheus.yml** et relancer le docker-compose

On doit rajouter ceci :

	- job_name: "Votre Job Docker"
	  static_configs:
	  	- targets: ["<YOUR  IP>:9323"]

  
A ce stade une nouvelle target devrait apparaître dans l'interface prometheus

  

###  4 - Création d'un dashbord simple de 3 métriques :
Ce dashboard affichera les métriques suivantes (avec les requêttes adéquates) : 
- Nombre de container **en cours d'exécution**
			 
		engine_daemon_container_states_containers{state="running"}

- Nombre de container **arrêté**

		engine_daemon_container_states_containers{state="stopped"}

- Nombre de container en **pause**

		engine_daemon_container_states_containers{state="paused"}

Configurez le dashboard et visualisez ls métriques.
