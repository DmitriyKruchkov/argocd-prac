# GitOps repo — пример структуры для Argo CD

## Структура

```
gitops-repo/
├── argocd/
│   ├── app-of-apps.yaml        # корневой Application, ставится руками один раз
│   └── applications/           # дочерние Application, подхватываются root автоматически
│       ├── myapp-dev.yaml
│       ├── myapp-staging.yaml
│       └── myapp-prod.yaml
└── apps/
    └── myapp/
        ├── base/                # общие манифесты (Kustomize base)
        │   ├── deployment.yaml
        │   ├── service.yaml
        │   └── kustomization.yaml
        └── overlays/            # различия по окружениям
            ├── dev/
            ├── staging/
            └── prod/
```

## Как развернуть

1. Замените `repoURL: https://github.com/DmitriyKruchkov/argocd-prac.git` на реальный URL вашего репозитория
   во всех Application-манифестах (`argocd/app-of-apps.yaml` и `argocd/applications/*.yaml`).

2. Запушьте этот репозиторий в git.

3. Примените только корневой Application:

   ```bash
   kubectl apply -f argocd/app-of-apps.yaml
   ```

4. Argo CD сам найдёт `argocd/applications/*.yaml` и создаст три дочерних
   Application (dev/staging/prod), каждая засинкает свой overlay в свой namespace.

## Как катить новую версию

```bash
cd gitops-repo
sed -i 's/newTag: v1.4.2/newTag: v1.5.0/' apps/myapp/overlays/prod/kustomization.yaml
git commit -am "bump myapp prod to v1.5.0"
git push
```

Argo CD увидит diff и засинкает автоматически (для dev/staging — `selfHeal: true`),
для prod синк ручным контролем — доводите через UI/CLI (`argocd app sync myapp-prod`),
т.к. `selfHeal: false`.

## Проверка локально без установки

Отрендерить манифесты, которые реально применятся, можно без Argo CD и без кластера:

```bash
kubectl kustomize apps/myapp/overlays/prod
```
