# Projet IC GROUP – Déploiement Odoo + pgAdmin + Site vitrine sur Kubernetes

## 📄 Description du projet
Ce projet consiste à déployer sur un cluster Kubernetes les services suivants pour la société **IC GROUP** :

- L'ERP **Odoo** (version communautaire 15)
- L'interface de gestion **pgAdmin** pour PostgreSQL
- Un **site vitrine** développé par l'équipe IC GROUP, permettant d'accéder aux deux applications via variables d'environnement

Le tout est orchestré dans un namespace Kubernetes `icgroup`, avec persistance des données via volumes, configuration des services via NodePort, et configuration automatique des connexions à la base de données.

---

## 🛠️ Technologies utilisées
- Kubernetes (Minikube / k3s)
- Docker (pour les images personnalisées)
- Odoo 15 (image officielle)
- PostgreSQL 13 (image officielle)
- pgAdmin4 (image officielle)
- Python + Flask (pour le site vitrine)

---

## 💲 Objectifs atteints

Cette section liste tous les objectifs visés dans le cadre du projet et coche ceux qui ont été atteints :

- [x] Conteneurisation du site vitrine Flask avec variables `ODOO_URL` et `PGADMIN_URL`
- [x] Construction de l'image Docker `ic-webapp:1.0`
- [x] Push sur Docker Hub
- [x] Déploiement d'Odoo et PostgreSQL avec persistance
- [x] Configuration automatique de pgAdmin via `servers.json`
- [x] Accès aux services via `NodePort`
- [x] Initialisation automatique de la base Odoo avec `-i base`

> 💡 Si cette liste n'apparaît pas correctement sur GitHub, vérifiez que le fichier est bien interprété comme Markdown (extension `.md`) et que vous êtes dans le bon affichage (pas l'affichage brut).

---

## 📖 Étapes détaillées du projet

### 1. Création du Dockerfile pour le site vitrine
Un fichier `Dockerfile` est écrit pour conteneuriser l'application Flask. Ce fichier contient :

```dockerfile
FROM python:3.6-alpine
WORKDIR /opt
COPY . /opt
RUN pip install flask==1.1.2
ENV ODOO_URL=https://www.odoo.com
ENV PGADMIN_URL=https://www.pgadmin.org
EXPOSE 8080
ENTRYPOINT ["python", "app.py"]
```

Et un fichier `requirements.txt` :
```
Flask
```

### 2. Build, test local, et push sur Docker Hub

#### Build de l'image
```bash
docker build -t ic-webapp:1.0 .
```

#### Test local avec variables d'environnement
```bash
docker run -d --name test-ic-webapp -p 8080:8080 -e ODOO_URL=https://www.odoo.com -e PGADMIN_URL=https://www.pgadmin.org ic-webapp:1.0
```

#### Suppression du conteneur de test
```bash
docker rm -f test-ic-webapp
```

#### Push vers Docker Hub
```bash
docker tag ic-webapp:1.0 seeb961/ic-webapp:1.0
docker push seeb961/ic-webapp:1.0
```

---

### 3. Création du namespace
On commence par isoler tous les composants du projet dans un namespace dédié :
```bash
kubectl create namespace icgroup
```
Cela permet de mieux gérer, lister et nettoyer les ressources plus tard.

---

### 4. Déploiement de PostgreSQL
PostgreSQL est le SGBD qui stocke les données Odoo. On crée un volume persistant pour garantir la conservation des données.
- L'utilisateur `odoo` est créé avec le mot de passe `odoo`
- La base `odoo` est créée automatiquement via l'environnement `POSTGRES_DB`

---

### 5. Configuration et déploiement de pgAdmin
pgAdmin est une interface web de gestion de PostgreSQL. On utilise un `ConfigMap` pour préconfigurer automatiquement la connexion à la base via un fichier `servers.json`.

---

### 6. Déploiement d'Odoo
L'application Odoo est initialisée via les arguments `-d odoo -i base` pour créer les tables essentielles de l'ERP. On monte un volume pour stocker les fichiers de l'application.
Un `initContainer` attend que PostgreSQL soit prêt avant de démarrer.

---

### 7. Exposition des services
Les services sont exposés via `NodePort` pour permettre l'accès externe depuis un navigateur. Les ports utilisés sont personnalisés (ex: 30069 pour Odoo).

---

### 8. Accès aux applications
Après déploiement, les services sont accessibles à l'adresse IP du nœud du cluster + le port :
- http://<IP>:30069 pour Odoo
- http://<IP>:30080 pour pgAdmin
- http://<IP>:30081 pour le site vitrine

---

## 📄 Rapport final
Toutes les applications sont fonctionnelles, accessibles via le site vitrine, avec stockage persistant, et configuration automatique des connexions PostgreSQL.

Réalisé par :

Sébastien GHZAYEL 5SRC2
