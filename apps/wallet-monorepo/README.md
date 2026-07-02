# Déploiement Skoleom Wallet sur Kubernetes

Architecture : un seul point d'entrée exposé (le client nginx), qui sert le front
et relaie `/api` vers le server. Server et Postgres restent internes (ClusterIP).

```
Internet ─▶ NodePort 30080 ─▶ [client nginx] ─(/api)─▶ [server :4000] ─▶ [postgres]
```

## Ordre de déploiement

```bash
# 1. Namespace
kubectl apply -f 00-namespace.yaml

# 2. Secret (impératif — les vraies valeurs ne touchent pas git)
kubectl create secret generic wallet-secrets -n wallet \
  --from-literal=DATABASE_URL='postgresql://skoleom:skoleom_dev@postgres:5432/wallet' \
  --from-literal=JWT_SECRET="$(openssl rand -hex 32)" \
  --from-literal=STRIPE_SECRET_KEY='sk_test_dummy' \
  --from-literal=STRIPE_WEBHOOK_SECRET='whsec_dummy'

# 3. Le reste (postgres, server, client)
kubectl apply -f .

# 4. Suivre le démarrage
kubectl get pods -n wallet -w
```

Une fois les 3 pods `Running` → l'app est sur `http://<IP_VPS>:30080`

## Notes
- Postgres utilise `emptyDir` → données **éphémères** (perdues au restart du pod).
  Pour persister : installer local-path-provisioner + PVC (exercice CKA).
- Le client embarque la conf nginx : si tu changes `client/nginx.conf`, rebuild + repush l'image client.
