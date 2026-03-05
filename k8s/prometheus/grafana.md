



## yaml配置
### env

#### 使用env配置grafana.ini 
为了支持k8s yaml配置grafana支持在env配置里面修改 grafana.ini配置，但是由于init和env配置的区别因此
在 Kubernetes 中部署 Grafana 时，如果你想通过 **环境变量** 的方式来配置以下两个字段：

```ini
[auth]
login_maximum_inactive_lifetime_days = 7
login_maximum_active_lifetime_days = 30
```

你可以使用 Grafana 提供的 **环境变量映射机制**。Grafana 支持将 `grafana.ini` 中的配置项通过环境变量进行覆盖。

---

✅ 正确的环境变量命名规则

Grafana 使用如下格式将 `.ini` 配置项转换为环境变量：

```
GF_<SectionName>_<KeyName>=<Value>
```

- Section 名（如 `[auth]`）转换为大写，去掉中括号
- Key 名（如 `login_maximum_inactive_lifetime_days`）也转换为大写
- 用下划线 `_` 替换点 `.` 和其他特殊字符

---

## 🔧 对应的环境变量是：

```yaml
- name: GF_AUTH_LOGIN_MAXIMUM_INACTIVE_LIFETIME_DAYS
  value: "7"
- name: GF_AUTH_LOGIN_MAXIMUM_ACTIVE_LIFETIME_DAYS
  value: "30"
```

> 在grafana中，环境变量的优先级大于grafana.ini中的配置，如果环境变量和grafana.ini中不同，优先环境变量生效。但是环境变量中的配置并不会真的覆盖掉grafana.ini的配置，因此，当环境变量中配置了对应变量，并不能直接从grafana.ini中直观的看到

#### 关闭匿名登录
关闭匿名登录要及时调整rotate的时间，否者默认情况下可能会导致大批量的rotate请求，请求url为 `/api/user/auth-tokens/rotate`

```yaml
# 关闭匿名登录
- name: GF_AUTH_ANONYMOUS_ENABLED
  value: "false"
# 在关闭匿名登录之后，这里需要及时设置Token rota的时间间隔
- name: GF_AUTH_TOKEN_ROTATION_INTERVAL_MINUTES
  value: "30"
- name: GF_AUTH_LOGIN_MAXIMUM_INACTIVE_LIFETIME_DURATION
  value: "7d"
- name: GF_AUTH_LOGIN_MAXIMUM_LIFETIME_DURATION
  value: "30d"
```

