Kyverno:
--------
    helm repo add kyverno https://kyverno.github.io/kyverno/
    helm repo update
    helm upgrade --install kyverno kyverno/kyverno -n kyverno --create-namespace --version 3.8.0 -f stack/kyverno/values.yaml

    Policy-Reporter
    ---------------
    helm repo add policy-reporter https://kyverno.github.io/policy-reporter
    helm repo update

    helm upgrade --install policy-reporter policy-reporter/policy-reporter --namespace policy-reporter --create-namespace --version 3.7.4 -f stack/kyverno/values-pr.yaml

    kubectl get policyreports -A
    kubectl get clusterpolicy

    kpr port-forward svc/policy-reporter-ui 8082:8080

        Probar un caso:
        ----------------
            cd cases/01-resource-requests-limits
            k apply -f policy.yaml
            k apply -f manifest-error.yaml
            k apply -f manifest-fixed.yaml
---

# Controles recurrentes con Kyverno
Aplicación base: microservices-demo en namespace `default`. Cada carpeta contiene manifiesto con error, manifiesto corregido y política Kyverno.
## Familias y ejemplos recurrentes
### Recursos y costos
- **Exigir requests y limits en Deployments** (`cases/01-resource-requests-limits`): Control recurrente para evitar scheduling impredecible, consumo sin límite y ruido en capacity planning/HPA.
### Pod Security / hardening
- **Bloquear privileged, escalación de privilegios y capabilities peligrosas** (`cases/02-container-security-context`): Ejemplo típico de controles Pod Security: privileged=false, allowPrivilegeEscalation=false, readOnlyRootFilesystem=true y drop ALL.
### Identidad y mínimo privilegio
- **Forzar ejecución como usuario no root** (`cases/03-run-as-non-root`): Control recurrente para reducir impacto ante compromiso del contenedor y alinearse a estándares Restricted.
### Supply chain / imágenes
- **Restringir registros permitidos y bloquear tag latest** (`cases/04-approved-images-no-latest`): Control recurrente para evitar imágenes no auditadas y despliegues no reproducibles.
### Disponibilidad / confiabilidad
- **Exigir readinessProbe y livenessProbe** (`cases/05-readiness-liveness-probes`): Control recurrente para evitar tráfico hacia pods no listos y mejorar recuperación automática ante fallas.
### Scheduling / aislamiento
- **Exigir nodeSelector.node=workloads** (`cases/06-node-selector-workloads`): Control recurrente para separar nodos de sistema y nodos de aplicaciones, evitando scheduling accidental.
### Gobernanza / metadata
- **Exigir labels de ownership y entorno** (`cases/07-ownership-labels`): Control recurrente para reportes, costos, ownership, soporte, auditoría y automatizaciones de plataforma.
### Networking / service mesh
- **Exigir appProtocol: grpc en Services gRPC** (`cases/08-grpc-service-appprotocol`): Control recurrente en service mesh y Gateway API para que el tráfico sea interpretado correctamente.
### Identidad / credenciales
- **Deshabilitar automount de token en ServiceAccounts** (`cases/09-serviceaccount-automount-false`): Control recurrente para reducir exposición de tokens cuando el workload no necesita hablar con la API de Kubernetes.
### Secrets / configuración sensible
- **Bloquear secretos en texto plano dentro de env.value** (`cases/10-no-plain-text-secrets-env`): Control recurrente para evitar credenciales hardcodeadas en manifiestos y repositorios GitOps.
### Observabilidad / estándares de plataforma
- **Exigir anotaciones OTel cuando el servicio declara otel_injection=enabled** (`cases/11-otel-annotations`): Control recurrente para que los servicios generados por el golden path mantengan trazas/métricas/logs instrumentados.
### Resiliencia / lifecycle
- **Evitar terminationGracePeriodSeconds demasiado bajo** (`cases/12-termination-grace-period`): Control recurrente para reducir cortes durante rolling updates, drenado y terminación de pods.

## Mapeo rápido de casos
| Caso | Familia | Qué controla | Carpeta |
|---:|---|---|---|
| 01 | Recursos y costos | Exigir requests y limits en Deployments | `cases/01-resource-requests-limits` |
| 02 | Pod Security / hardening | Bloquear privileged, escalación de privilegios y capabilities peligrosas | `cases/02-container-security-context` |
| 03 | Identidad y mínimo privilegio | Forzar ejecución como usuario no root | `cases/03-run-as-non-root` |
| 04 | Supply chain / imágenes | Restringir registros permitidos y bloquear tag latest | `cases/04-approved-images-no-latest` |
| 05 | Disponibilidad / confiabilidad | Exigir readinessProbe y livenessProbe | `cases/05-readiness-liveness-probes` |
| 06 | Scheduling / aislamiento | Exigir nodeSelector.node=workloads | `cases/06-node-selector-workloads` |
| 07 | Gobernanza / metadata | Exigir labels de ownership y entorno | `cases/07-ownership-labels` |
| 08 | Networking / service mesh | Exigir appProtocol: grpc en Services gRPC | `cases/08-grpc-service-appprotocol` |
| 09 | Identidad / credenciales | Deshabilitar automount de token en ServiceAccounts | `cases/09-serviceaccount-automount-false` |
| 10 | Secrets / configuración sensible | Bloquear secretos en texto plano dentro de env.value | `cases/10-no-plain-text-secrets-env` |
| 11 | Observabilidad / estándares de plataforma | Exigir anotaciones OTel cuando el servicio declara otel_injection=enabled | `cases/11-otel-annotations` |
| 12 | Resiliencia / lifecycle | Evitar terminationGracePeriodSeconds demasiado bajo | `cases/12-termination-grace-period` |
