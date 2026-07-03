# K8s — Module Auth

## Vue d'ensemble

Le module `auth` est un service NestJS **stateless** (pas de base de données, uniquement des opérations JWT). Cette caractéristique le rend parfait pour Kubernetes : chaque réplique est identique, le scale horizontal est sans friction, et un redémarrage de pod n'entraîne aucune perte d'état.

Ce qui est mis en place :
- Déploiement Kubernetes dans le namespace `urbanflow`
- Image Docker publiée sur `ghcr.io/urbanflow-mns/urbanflow-auth:latest`
- Autoscaling automatique (HPA) entre 1 et 4 répliques selon la charge CPU
- Pipeline CI/CD GitHub Actions qui rebuilde et redéploie à chaque push sur `modules/auth`

---

## Architecture

```
Client HTTP
    │
    ▼
[ gateway-service :4000 ]  ←── seul point d'entrée HTTP
    │
    ├── RabbitMQ (AUTH_QUEUE) ──────────────────────────────┐
    │                                                        ▼
    └── TCP :6001 ──────────────────────────────► [ auth-service K8s Service ]
                                                      │
                                          ┌───────────┴───────────┐
                                          ▼                       ▼
                                   [ auth-pod-1 ]         [ auth-pod-2 ]
                                  (NestJS :4001)         (NestJS :4001)
                                          │                       │
                                          └───────────┬───────────┘
                                                      ▼
                                             RabbitMQ (AMQP)
                                           ├── LOGS_QUEUE
                                           └── NOTIFICATIONS_QUEUE
                                                      +
                                             user-service TCP :6006
```

Le Service Kubernetes assure le round-robin entre les pods. Le gateway n'a pas connaissance du nombre de répliques.

---

## Manifests K8s

Tous les fichiers se trouvent dans `k8s/auth/`.

| Fichier | Rôle |
|---|---|
| `namespace.yaml` | Crée le namespace `urbanflow` isolant tous les services UrbanFlow |
| `configmap.yaml` | Variables d'environnement non sensibles (`API_PORT`, `TCP_PORT`, hôtes RabbitMQ, etc.) |
| `secret.example.yaml` | Exemple de Secret K8s pour les variables sensibles (`JWT_SECRET`, credentials RabbitMQ). À ne **jamais** appliquer tel quel — copier et remplir manuellement |
| `deployment.yaml` | Définit les pods auth : image, ports (4001 HTTP + 6001 TCP), ressources CPU/mémoire, référence ConfigMap et Secret |
| `service.yaml` | Expose le Deployment en interne au cluster (ClusterIP) sur les ports 4001 et 6001 |
| `hpa.yaml` | HorizontalPodAutoscaler : 1 à 4 répliques, seuil de déclenchement à 50% CPU |

---

## Autoscaling (HPA)

```
metrics-server (agrège CPU pods)
    │
    ▼
HorizontalPodAutoscaler
  minReplicas: 1 / maxReplicas: 4 / targetCPUUtilization: 50%
    │
    ├── CPU < 50% → scale down (jusqu'à 1 réplique)
    └── CPU > 50% → scale up (jusqu'à 4 répliques)
          │
          ▼
   Deployment (auth-service)
     pods créés/détruits automatiquement
          │
          ▼
   Service (round-robin entre pods actifs)
```

Le module auth est **stateless** : chaque réplique peut traiter n'importe quelle requête indépendamment. Pas de session à synchroniser, pas de cache à partager. Le scale-out est donc transparent.

`metrics-server` doit être activé sur le cluster (addon minikube ou installé sur k3s).

---

## Commandes utiles

| Commande | Description |
|---|---|
| `kubectl get pods -n urbanflow` | Lister les pods et leur statut |
| `kubectl get hpa -n urbanflow` | Voir l'état de l'autoscaler (replicas, CPU actuel/cible) |
| `kubectl get hpa -n urbanflow -w` | Surveiller le HPA en temps réel |
| `kubectl top pods -n urbanflow` | Consommation CPU/mémoire des pods |
| `kubectl logs -f <pod> -n urbanflow` | Logs d'un pod en temps réel |
| `kubectl logs -f -l app=auth-service -n urbanflow` | Logs de tous les pods auth |
| `kubectl rollout restart deployment/auth-service -n urbanflow` | Forcer un redéploiement rolling |
| `kubectl rollout status deployment/auth-service -n urbanflow` | Suivre l'avancement d'un déploiement |
| `kubectl describe hpa auth-hpa -n urbanflow` | Détail des événements HPA (scale-up/down) |
| `kubectl describe pod <pod> -n urbanflow` | Diagnostiquer un pod (events, ressources) |
| `kubectl get secret auth-secrets -n urbanflow` | Vérifier que le secret existe |
| `kubectl delete pod <pod> -n urbanflow` | Forcer la recréation d'un pod |
