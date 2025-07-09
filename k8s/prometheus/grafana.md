











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

