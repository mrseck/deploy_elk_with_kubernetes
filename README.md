# Stack ELK sur Kubernetes

Ce projet déploie une stack ELK (Elasticsearch et Kibana) sur Kubernetes en utilisant l'opérateur Elastic Cloud on Kubernetes (ECK).

## 📋 Description

Cette configuration permet de déployer facilement Elasticsearch et Kibana sur un cluster Kubernetes. Le déploiement utilise l'opérateur ECK qui simplifie la gestion et l'orchestration des ressources Elasticsearch et Kibana dans Kubernetes.

## 🏗️ Architecture

Le projet déploie :

- **Elasticsearch** : Cluster avec 3 nœuds (version 9.1.6)
  - Chaque nœud joue les rôles : master, data et ingest
  - Stockage persistant de 30Gi par nœud
  - Ressources : 500Mi-1Gi RAM, 500m-1 CPU

- **Kibana** : Interface de visualisation (version 9.1.6)
  - 1 instance
  - Connectée au cluster Elasticsearch nommé "quickstart"

## 📦 Prérequis

Avant de déployer cette stack, assurez-vous d'avoir :

1. **Un cluster Kubernetes** fonctionnel (version 1.19+)
2. **L'opérateur ECK installé** dans votre cluster
   ```bash
   kubectl create -f https://download.elastic.co/downloads/eck/3.0.0/crds.yaml
   kubectl apply -f https://download.elastic.co/downloads/eck/3.0.0/operator.yaml
   ```
3. **kubectl** configuré pour accéder à votre cluster
4. **Permissions suffisantes** pour créer des ressources dans le namespace cible

## 🚀 Déploiement

### Étape 1 : Vérifier l'opérateur ECK

Vérifiez que l'opérateur ECK est installé et fonctionnel :

```bash
kubectl get pods -n elastic-system
```

### Étape 2 : Déployer Elasticsearch

Déployez le cluster Elasticsearch :

```bash
kubectl apply -f elasticsearch.yaml
```

Attendez que le cluster soit prêt :

```bash
kubectl get elasticsearch
kubectl get pods -l elasticsearch.k8s.elastic.co/cluster-name=elasticsearch
```

### Étape 3 : Déployer Kibana

Une fois Elasticsearch opérationnel, déployez Kibana :

```bash
kubectl apply -f kibana.yaml
```

Vérifiez le statut de Kibana :

```bash
kubectl get kibana
kubectl get pods -l kibana.k8s.elastic.co/name=kibana
```

## 🔍 Vérification du déploiement

### Vérifier l'état des ressources

```bash
# Vérifier Elasticsearch
kubectl get elasticsearch elasticsearch

# Vérifier Kibana
kubectl get kibana kibana

# Vérifier les pods
kubectl get pods -l elasticsearch.k8s.elastic.co/cluster-name=elasticsearch
kubectl get pods -l kibana.k8s.elastic.co/name=kibana
```

### Vérifier les services

```bash
# Lister les services Elasticsearch
kubectl get svc -l elasticsearch.k8s.elastic.co/cluster-name=elasticsearch

# Lister les services Kibana
kubectl get svc -l kibana.k8s.elastic.co/name=kibana
```

## 🔐 Accès aux services

### Récupérer les identifiants par défaut

Les identifiants par défaut sont stockés dans un secret Kubernetes :

```bash
# Pour Elasticsearch
kubectl get secret elasticsearch-es-elastic-user -o jsonpath="{.data.elastic}" | base64 -d
echo ""

# Pour Kibana (les identifiants sont les mêmes)
```

### Accéder à Kibana

1. **Port-forward** (pour un accès local) :
   ```bash
   kubectl port-forward service/kibana-kb-http 5601:5601
   ```
   Puis accédez à : https://localhost:5601

2. **Via Ingress** (si configuré) :
   Accédez à l'URL de l'ingress configuré pour Kibana

3. **Via NodePort/LoadBalancer** :
   Si un service de type NodePort ou LoadBalancer est configuré, utilisez l'IP/port exposé

## 📊 Configuration détaillée

### Elasticsearch

- **Version** : 9.1.6
- **Nombre de nœuds** : 3
- **Rôles des nœuds** : master, data, ingest
- **Ressources par nœud** :
  - Mémoire : 500Mi (requis) / 1Gi (limite)
  - CPU : 500m (requis) / 1 (limite)
- **Stockage** : 30Gi par nœud (ReadWriteOnce)

### Kibana

- **Version** : 9.1.6
- **Nombre d'instances** : 1
- **Référence Elasticsearch** : quickstart (note : vérifiez que le nom correspond à votre cluster Elasticsearch)

## ⚠️ Notes importantes

1. **Nom du cluster Elasticsearch** : Le fichier `kibana.yaml` référence un cluster Elasticsearch nommé "quickstart", mais le fichier `elasticsearch.yaml` crée un cluster nommé "elasticsearch". Vous devrez soit :
   - Modifier le nom dans `elasticsearch.yaml` pour "quickstart", ou
   - Modifier la référence dans `kibana.yaml` pour "elasticsearch"

2. **Stockage** : Assurez-vous que votre cluster Kubernetes supporte les PersistentVolumes et que vous avez suffisamment d'espace de stockage (90Gi au total pour les 3 nœuds).

3. **Ressources** : Vérifiez que votre cluster a suffisamment de ressources CPU et mémoire pour supporter le déploiement.

4. **Sécurité** : En production, configurez :
   - TLS/SSL pour les communications
   - Authentification et autorisation appropriées
   - Sauvegarde régulière des données
   - Monitoring et alerting

## 🔧 Dépannage

### Les pods ne démarrent pas

```bash
# Vérifier les événements
kubectl describe pod <nom-du-pod>

# Vérifier les logs
kubectl logs <nom-du-pod>
```

### Problèmes de stockage

```bash
# Vérifier les PersistentVolumeClaims
kubectl get pvc

# Vérifier les PersistentVolumes
kubectl get pv
```

### Problèmes de connexion Kibana -> Elasticsearch

Vérifiez que :
1. Le cluster Elasticsearch est en état "Ready"
2. Le nom de référence dans `kibana.yaml` correspond au nom du cluster Elasticsearch
3. Les secrets sont correctement créés

## 🗑️ Suppression

Pour supprimer le déploiement :

```bash
# Supprimer Kibana
kubectl delete -f kibana.yaml

# Supprimer Elasticsearch
kubectl delete -f elasticsearch.yaml

# Note : Les PersistentVolumeClaims ne seront pas supprimés automatiquement
# Supprimez-les manuellement si nécessaire :
kubectl delete pvc -l elasticsearch.k8s.elastic.co/cluster-name=elasticsearch
```

## 📚 Ressources supplémentaires

- [Documentation ECK](https://www.elastic.co/guide/en/cloud-on-k8s/current/index.html)
- [Documentation Elasticsearch](https://www.elastic.co/guide/en/elasticsearch/reference/9.1/index.html)
- [Documentation Kibana](https://www.elastic.co/guide/en/kibana/9.1/index.html)

## 📝 Licence

Ce projet est fourni à titre d'exemple et peut être utilisé librement.

