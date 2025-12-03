# Jenkins CI/CD avec React + TypeScript + Vite

Projet de démonstration d'intégration continue avec Jenkins pour une application React TypeScript construite avec Vite, optimisé pour Ubuntu Linux.

## 🚀 Technologies

- **Frontend**: React 19.2.0 + TypeScript 5.9.3
- **Build Tool**: Vite 7.2.4
- **CI/CD**: Jenkins Pipeline
- **Linting**: ESLint 9.39.1
- **OS**: Ubuntu Linux

## 📋 Prérequis Ubuntu

### Installation Node.js
```bash
# Mise à jour du système
sudo apt update && sudo apt upgrade -y

# Installation Node.js via NodeSource
curl -fsSL https://deb.nodesource.com/setup_20.x | sudo -E bash -
sudo apt-get install -y nodejs

# Vérification
node --version
npm --version
```

### Installation Jenkins sur Ubuntu

#### Option 1: Jenkins avec Docker (Recommandé)
```bash
# Installation Docker
sudo apt update
sudo apt install -y docker.io
sudo systemctl enable --now docker
sudo usermod -aG docker $USER
newgrp docker

# Lancement Jenkins
docker pull jenkins/jenkins:lts-jdk21
docker volume create jenkins_home

docker run -d \
  --name jenkins \
  -p 8080:8080 \
  -p 50000:50000 \
  --restart=on-failure \
  -v jenkins_home:/var/jenkins_home \
  -v /var/run/docker.sock:/var/run/docker.sock \
  jenkins/jenkins:lts-jdk21

# Récupération du mot de passe initial
docker exec jenkins cat /var/jenkins_home/secrets/initialAdminPassword

# Accès: http://localhost:8080
```

#### Option 2: Installation Native
```bash
# Installation Java
sudo apt install openjdk-11-jdk -y

# Ajout du repository Jenkins
wget -q -O - https://pkg.jenkins.io/debian-stable/jenkins.io.key | sudo apt-key add -
sudo sh -c 'echo deb https://pkg.jenkins.io/debian-stable binary/ > /etc/apt/sources.list.d/jenkins.list'

# Installation Jenkins
sudo apt update
sudo apt install jenkins -y

# Démarrage du service
sudo systemctl start jenkins
sudo systemctl enable jenkins

# Accès: http://localhost:8080
sudo cat /var/lib/jenkins/secrets/initialAdminPassword
```

## 🛠️ Installation du Projet

```bash
# Clonage du repository
git clone https://github.com/Keltoummalouki/Jenkins-doc.git
cd jenkins-doc

# Installation des dépendances
npm install

# Démarrage en développement
npm run dev
```

## 📜 Scripts Disponibles

```bash
npm run dev      # Serveur de développement (http://localhost:5173)
npm run build    # Build de production
npm run lint     # Vérification ESLint
npm run preview  # Aperçu du build
```

## 🔧 Configuration Jenkins

### 1. Création du Job Jenkins
```
# Dans Jenkins Dashboard
Nouveau Job → Pipeline → jenkins-doc
```

### 2. Configuration SCM
```
Repository URL: https://github.com/Keltoummalouki/Jenkins-doc
Branch: */main
Script Path: Jenkinsfile
```

### 3. Plugins Jenkins Requis
```
# Installation via Jenkins CLI ou Interface
- Git Plugin
- Pipeline Plugin
- NodeJS Plugin (optionnel)
- Workspace Cleanup Plugin
- Docker Pipeline Plugin (si utilisation Docker)
```

### 4. Configuration Docker dans Jenkins (pour pipeline Docker)
```bash
# Installer Docker dans le container Jenkins
docker exec -u root jenkins bash -c "
  apt-get update && \
  apt-get install -y docker.io && \
  usermod -aG docker jenkins
"

# Redémarrer Jenkins
docker restart jenkins
```

## 🚀 Pipeline Jenkins

Le `Jenkinsfile` contient les étapes suivantes :

1. **Checkout** - Récupération du code source
2. **Check Environment** - Vérification de l'environnement
3. **Mock Install Dependencies** - Simulation installation des dépendances
4. **Mock Lint** - Simulation vérification du code
5. **Mock Test** - Simulation des tests
6. **Mock Build** - Simulation construction de l'application
7. **Archive Artifacts** - Archivage des fichiers de build

### Pipeline Réel (Option A: Docker Agent)

Pour activer un pipeline réel avec Node.js, remplacez le `Jenkinsfile` par:

```groovy
pipeline {
    agent {
        docker {
            image 'node:20-alpine'
            args '-v $PWD:/workspace -w /workspace'
        }
    }

    options {
        timestamps()
        ansiColor('xterm')
    }

    environment {
        CI = 'true'
    }

    stages {
        stage('Checkout') {
            steps {
                echo 'Cloning Repository...'
                checkout scm
            }
        }

        stage('Check Environment') {
            steps {
                sh 'node -v && npm -v'
                sh 'pwd && ls -la'
            }
        }

        stage('Install Dependencies') {
            steps {
                sh '''
                    if [ -f package-lock.json ]; then
                        npm ci
                    else
                        npm install
                    fi
                '''
            }
        }

        stage('Lint') {
            steps {
                sh 'npm run lint'
            }
        }

        stage('Build') {
            steps {
                sh 'npm run build'
            }
        }

        stage('Archive Artifacts') {
            steps {
                archiveArtifacts artifacts: 'dist/**', fingerprint: true
            }
        }
    }

    post {
        always {
            cleanWs()
        }
        success {
            echo '✅ Pipeline completed successfully!'
        }
        failure {
            echo '❌ Pipeline failed!'
        }
    }
}
```

### Pipeline Réel (Option B: NodeJS Plugin)

1. Installer le plugin NodeJS dans Jenkins
2. Configurer l'outil Node.js: **Manage Jenkins** → **Tools** → **NodeJS installations**
3. Utiliser ce Jenkinsfile:

```groovy
pipeline {
    agent any

    tools {
        nodejs 'Node 20'  // Nom configuré dans Jenkins
    }

    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Install') {
            steps {
                sh 'npm ci'
            }
        }

        stage('Lint') {
            steps {
                sh 'npm run lint'
            }
        }

        stage('Build') {
            steps {
                sh 'npm run build'
            }
        }

        stage('Archive') {
            steps {
                archiveArtifacts artifacts: 'dist/**', fingerprint: true
            }
        }
    }

    post {
        always { cleanWs() }
        success { echo '✅ Build réussi!' }
        failure { echo '❌ Build échoué!' }
    }
}
```

## 📁 Structure du Projet

```
jenkins-doc/
├── src/
│   ├── App.tsx          # Composant principal
│   ├── App.css          # Styles de l'app
│   ├── main.tsx         # Point d'entrée
│   ├── index.css        # Styles globaux
│   └── assets/          # Ressources statiques
├── dist/                # Build de production
├── Jenkinsfile          # Pipeline CI/CD
├── package.json         # Dépendances et scripts
├── tsconfig.json        # Configuration TypeScript
├── vite.config.ts       # Configuration Vite
└── eslint.config.js     # Configuration ESLint
```

## 🔍 Monitoring Jenkins

### Logs en Temps Réel

```bash
# Logs Docker Jenkins
docker logs -f jenkins

# Logs Jenkins natif
sudo tail -f /var/log/jenkins/jenkins.log

# Logs du build
# Accessible via Jenkins UI → Job → Console Output
```

### Commandes Utiles Ubuntu

#### Docker Jenkins
```bash
# Status container
docker ps -a | grep jenkins

# Redémarrage Jenkins
docker restart jenkins

# Arrêt Jenkins
docker stop jenkins

# Démarrage Jenkins
docker start jenkins

# Supprimer container (données conservées dans volume)
docker rm jenkins
```

#### Jenkins Natif
```bash
# Status Jenkins
sudo systemctl status jenkins

# Redémarrage Jenkins
sudo systemctl restart jenkins

# Arrêt Jenkins
sudo systemctl stop jenkins

# Logs
sudo journalctl -u jenkins -f
```

#### Système
```bash
# Espace disque
df -h

# Processus Node.js
ps aux | grep node

# Ports utilisés
sudo netstat -tlnp | grep :8080
```

## 🐛 Dépannage

### Erreurs Communes

**Permission denied sur Ubuntu (Jenkins natif):**
```bash
sudo chown -R jenkins:jenkins /var/lib/jenkins/workspace/
sudo chmod +x /var/lib/jenkins/workspace/jenkins-doc/
```

**Permission denied Docker:**
```bash
# Ajouter l'utilisateur au groupe docker
sudo usermod -aG docker $USER
newgrp docker
```

**Node.js non trouvé:**
```bash
# Configuration PATH dans Jenkins
Manage Jenkins → Global Tool Configuration → NodeJS
```

**Port 8080 occupé:**
```bash
# Identifier le processus
sudo netstat -tlnp | grep :8080

# Arrêter Jenkins
sudo systemctl stop jenkins  # ou docker stop jenkins

# Changer de port (modifier docker run avec -p 9090:8080)
```

**Erreur "Tool type nodejs does not have an install":**
- Utiliser l'option Docker agent dans le Jenkinsfile
- OU installer et configurer le plugin NodeJS avec le bon nom d'outil

## 📊 Métriques de Build

- **Temps de build moyen**: ~2-3 minutes
- **Taille du bundle**: ~150KB (gzipped)
- **Tests**: Simulation (à implémenter)
- **Coverage**: À configurer

## 🔐 Sécurité

### Bonnes Pratiques Jenkins

- Utilisation de credentials Jenkins pour les secrets
- Restriction des permissions utilisateurs
- Mise à jour régulière des plugins
- Sauvegarde de la configuration Jenkins
- Utilisation de HTTPS en production

### Configuration Firewall Ubuntu

```bash
sudo ufw allow 8080/tcp  # Jenkins
sudo ufw allow 5173/tcp  # Vite dev server
sudo ufw enable
sudo ufw status
```

## 📚 Ressources

- [Documentation Jenkins](https://www.jenkins.io/doc/)
- [Guide Vite](https://vitejs.dev/guide/)
- [React TypeScript](https://react-typescript-cheatsheet.netlify.app/)
- [Ubuntu Server Guide](https://ubuntu.com/server/docs)
- [Docker Documentation](https://docs.docker.com/)

## 🤝 Contribution

1. Fork le projet
2. Créer une branche feature (`git checkout -b feature/nouvelle-fonctionnalite`)
3. Commit les changements (`git commit -m 'Ajout nouvelle fonctionnalité'`)
4. Push vers la branche (`git push origin feature/nouvelle-fonctionnalite`)
5. Ouvrir une Pull Request

## 👤 Auteur

**Keltoum Malouki**

- GitHub: [@Keltoummalouki](https://github.com/Keltoummalouki)
- Repository: [Jenkins-doc](https://github.com/Keltoummalouki/Jenkins-doc)

## 📄 Licence

Ce projet est sous licence MIT. Voir le fichier `LICENSE` pour plus de détails.

---

Développé avec ❤️ sur Ubuntu Linux
