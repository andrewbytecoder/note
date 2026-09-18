

## 命令


### 用户管理
1. 获取用户
```bash
$ oc get user
NAME             UID                                    FULL NAME   IDENTITIES
admin            3f6dff50-6763-4704-b81b-97151f0deb60               htpasswd:admin
hytera-monitor   c6de0c16-96fe-470f-925e-5e50bebc58ff               htpasswd:hytera-monitor
hytera-user      9cba127e-ebdd-409b-9c9e-703d0ae1062d               htpasswd:hytera-user
mcs-user         6846f942-422b-429b-bb27-a75f04acd598               htpasswd:mcs-user
```




### route 管理
1. 获取route
```bash
$ oc get route
NAME            HOST/PORT                     PATH   SERVICES   PORT   TERMINATION            WILDCARD
route-grafana   grafana.apps.z2.ameidc2.com          grafana    http   passthrough/Redirect   None
```