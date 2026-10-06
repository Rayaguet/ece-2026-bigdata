# Réponses – Lab 2 (Object storage S3)

## Pourquoi les identifiants sont dans un Secret et pas dans le ConfigMap ?
Un ConfigMap est stocké et affiché en clair, lisible par tous ceux qui ont accès au namespace.
Un Secret a des droits d'accès (RBAC) séparés, peut être chiffré au repos dans etcd et n'est pas affiché par `kubectl describe`.

## Les identifiants Onyxia sont temporaires : que se passe-t-il si le Job est relancé demain ?
Le Job échoue avec une erreur `ExpiredToken`, car le Secret contient des identifiants expirés.
En production, on donne au Job une identité liée à son ServiceAccount (IRSA / Workload Identity via OIDC),
ou on utilise un gestionnaire de secrets (Vault, External Secrets) qui renouvelle les identifiants automatiquement.

## Comment transformer ce Job en ingestion quotidienne ?
En le transformant en `CronJob` avec par exemple `schedule: "0 2 * * *"` (tous les jours à 2h),
et en écrivant dans un préfixe daté (`bronze/date=YYYY-MM-DD/`) pour ne pas écraser les données de la veille.

## Remarques
- Le Job était bloqué par le ResourceQuota `onyxia-quota` (« must specify limits.cpu »).
  J'ai ajouté `cpu: 200m` dans `resources.limits`.
- Politique de bucket (Deny DeleteObject sur bronze/) : la suppression de `protected.csv` a été ACCEPTÉE / REFUSÉE.
