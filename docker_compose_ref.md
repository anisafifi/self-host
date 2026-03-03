# 🐳 Docker Compose — Complete Key Reference
> Based on the **Compose Specification** (current standard, replaces v2/v3 versioning).  
> File names accepted: `compose.yaml` *(preferred)*, `compose.yml`, `docker-compose.yaml`, `docker-compose.yml`

---

## 📑 Table of Contents
1. [Top-Level Keys](#1-top-level-keys)
2. [services.*](#2-services)
   - [build.*](#21-build-sub-keys)
   - [deploy.*](#22-deploy-sub-keys)
   - [healthcheck.*](#23-healthcheck-sub-keys)
   - [logging.*](#24-logging-sub-keys)
   - [volumes (service-level)](#25-service-level-volumes-long-syntax)
   - [networks (service-level)](#26-service-level-networks-long-syntax)
   - [depends_on (long syntax)](#27-depends_on-long-syntax)
   - [secrets (service-level)](#28-service-level-secrets-long-syntax)
   - [configs (service-level)](#29-service-level-configs-long-syntax)
   - [ulimits.*](#210-ulimits)
   - [blkio_config.*](#211-blkio_config)
   - [extends.*](#212-extends)
3. [networks.*](#3-top-level-networks)
4. [volumes.*](#4-top-level-volumes)
5. [secrets.*](#5-top-level-secrets)
6. [configs.*](#6-top-level-configs)
7. [include.*](#7-include)
8. [Variable Interpolation & .env](#8-variable-interpolation--env)
9. [Fragments & Anchors (YAML)](#9-yaml-anchors--fragments)
10. [Watch Mode (Compose Watch)](#10-watch-mode-compose-watch)
11. [Full Annotated Example](#11-full-annotated-example)

---

## 1. Top-Level Keys

| Key | Description |
|-----|-------------|
| `name` | Sets the project name. Overrides the directory-based default. Must be lowercase alphanumeric + `-` + `_`. |
| `services` | **Required.** Map of all service definitions (containers). |
| `networks` | Top-level named networks shared across services. |
| `volumes` | Top-level named volumes shared across services. |
| `secrets` | Top-level secrets (files or environment values mounted securely). |
| `configs` | Top-level configs (non-sensitive configuration data). |
| `include` | List of other Compose files to merge into the current one. |
| `version` | **Deprecated.** Previously required (e.g., `"3.8"`). Now ignored by the Compose Specification. |

---

## 2. `services`

Each key under `services` is a **service name** (e.g., `web`, `db`). Its value is a service definition.

```yaml
services:
  <service-name>:
    # ... all keys below
```

### Core Identity

| Key | Type | Description |
|-----|------|-------------|
| `image` | string | Docker image to use (e.g., `nginx:latest`, `myapp:1.0`). Required if `build` is not set. |
| `build` | string \| map | Build context path or detailed build config. See [§2.1](#21-build-sub-keys). |
| `container_name` | string | Custom container name. Prevents scaling to multiple replicas if set. |
| `hostname` | string | Hostname inside the container. Defaults to the service name. |
| `domainname` | string | Custom domain name for the container. |

### Runtime Command & Entrypoint

| Key | Type | Description |
|-----|------|-------------|
| `command` | string \| list | Overrides the default `CMD` in the image. |
| `entrypoint` | string \| list | Overrides the `ENTRYPOINT` in the image. Set to `[]` to clear. |

### Environment

| Key | Type | Description |
|-----|------|-------------|
| `environment` | list \| map | Set environment variables inside the container. Values can be omitted to inherit from host shell. |
| `env_file` | string \| list \| map | Load env vars from a file. Long form supports `path` and `format` (`raw` or default). |

```yaml
# Short
env_file: .env

# List
env_file:
  - .env
  - ./secrets.env

# Long (with format)
env_file:
  - path: ./default.env
    required: true
  - path: ./override.env
    required: false
    format: raw
```

### Ports

| Key | Type | Description |
|-----|------|-------------|
| `ports` | list | Publish ports to the host. Short: `"HOST:CONTAINER"`. Long form supports `target`, `published`, `protocol`, `mode`, `host_ip`, `app_protocol`. |
| `expose` | list | Expose ports to linked services only (not to the host). |

```yaml
# Short syntax
ports:
  - "8080:80"
  - "127.0.0.1:5000:5000"
  - "3000"          # random host port

# Long syntax
ports:
  - target: 80
    published: "8080"
    host_ip: "127.0.0.1"
    protocol: tcp
    mode: host
    app_protocol: http
```

### Volumes & Bind Mounts

| Key | Type | Description |
|-----|------|-------------|
| `volumes` | list | Mount host paths, named volumes, or tmpfs into the container. See [§2.5](#25-service-level-volumes-long-syntax). |
| `volumes_from` | list | Mount all volumes from another service or container. Syntax: `service:ro` or `container:name:rw`. |
| `tmpfs` | string \| list | Mount a temporary in-memory filesystem. |

### Networking

| Key | Type | Description |
|-----|------|-------------|
| `networks` | list \| map | Networks this service connects to. See [§2.6](#26-service-level-networks-long-syntax). |
| `network_mode` | string | Set to `host`, `none`, `bridge`, or `service:<name>` or `container:<name>`. |
| `dns` | string \| list | Custom DNS servers. |
| `dns_search` | string \| list | DNS search domains. |
| `dns_opt` | list | DNS resolver options (e.g., `ndots:5`). |
| `extra_hosts` | list | Add entries to `/etc/hosts`. Format: `"hostname:IP"` or `"hostname=IP"`. |
| `links` | list | *Legacy.* Link to another service. Format: `service` or `service:alias`. |
| `external_links` | list | Link to containers started outside this Compose project. |

### Dependencies & Startup Order

| Key | Type | Description |
|-----|------|-------------|
| `depends_on` | list \| map | Declare service dependencies. Long form allows `condition` and `restart` options. See [§2.7](#27-depends_on-long-syntax). |

### Resource & Process Control

| Key | Type | Description |
|-----|------|-------------|
| `deploy` | map | Deployment configuration (replicas, resources, restart policies, placement). See [§2.2](#22-deploy-sub-keys). |
| `restart` | string | Restart policy: `no` (default), `always`, `on-failure`, `unless-stopped`. |
| `stop_signal` | string | Signal to send to stop the container (default: `SIGTERM`). |
| `stop_grace_period` | duration | Time to wait before SIGKILL after stop signal (e.g., `10s`, `1m30s`). |
| `init` | bool | Run an init process (PID 1) inside the container to handle zombie reaping. |
| `pid` | string | Set the PID namespace: `host` or `service:<name>`. |
| `ipc` | string | Set the IPC namespace: `shareable`, `host`, `service:<name>`, or `container:<name>`. |
| `shm_size` | string | Size of `/dev/shm` (e.g., `128m`). |
| `mem_limit` | string | *Legacy v2.* Memory limit (e.g., `512m`). Prefer `deploy.resources.limits.memory`. |
| `memswap_limit` | string | *Legacy v2.* Memory + swap limit. |
| `mem_reservation` | string | *Legacy v2.* Memory soft limit. |
| `cpu_shares` | int | *Legacy v2.* Relative CPU weight. |
| `cpu_period` | int | *Legacy v2.* CPU CFS period in microseconds. |
| `cpu_quota` | int | *Legacy v2.* CPU CFS quota in microseconds. |
| `cpus` | float | *Legacy v2.* Number of CPU cores (e.g., `0.5`). |
| `cpuset` | string | *Legacy v2.* CPUs to use (e.g., `"0,1"` or `"0-3"`). |
| `oom_kill_disable` | bool | Disable OOM killer for this container. |
| `oom_score_adj` | int | OOM score adjustment (-1000 to 1000). |

### Capabilities & Security

| Key | Type | Description |
|-----|------|-------------|
| `cap_add` | list | Add Linux capabilities (e.g., `NET_ADMIN`, `SYS_TIME`). |
| `cap_drop` | list | Drop Linux capabilities. |
| `privileged` | bool | Run container in privileged mode (full host access). Use with caution. |
| `security_opt` | list | Override default security options (e.g., `no-new-privileges`, `label:disable`, `apparmor:profile`). |
| `sysctls` | list \| map | Set kernel parameters inside the container (e.g., `net.core.somaxconn=1024`). |
| `read_only` | bool | Mount the root filesystem as read-only. |
| `userns_mode` | string | User namespace mode (e.g., `host`). |

### User & Group

| Key | Type | Description |
|-----|------|-------------|
| `user` | string | Override the user to run as (e.g., `"1000"`, `"user:group"`). |
| `group_add` | list | Add supplementary groups to the container's user. |
| `working_dir` | string | Override the working directory inside the container. |

### Device & Hardware Access

| Key | Type | Description |
|-----|------|-------------|
| `devices` | list | Mount host devices into the container. Format: `"HOST_PATH:CONTAINER_PATH:OPTIONS"`. |
| `device_cgroup_rules` | list | Add custom cgroup device rules. Format: `"TYPE MAJOR:MINOR ACCESS"`. |

### Labels & Annotations

| Key | Type | Description |
|-----|------|-------------|
| `labels` | list \| map | Add metadata labels to the container. |
| `annotations` | list \| map | Add OCI annotations to the container image metadata. |

### Healthcheck

| Key | Type | Description |
|-----|------|-------------|
| `healthcheck` | map | Configures a health check for the container. See [§2.3](#23-healthcheck-sub-keys). |

### Logging

| Key | Type | Description |
|-----|------|-------------|
| `logging` | map | Configure log driver and options. See [§2.4](#24-logging-sub-keys). |

### Secrets & Configs

| Key | Type | Description |
|-----|------|-------------|
| `secrets` | list \| map | Grant access to secrets defined at top-level. See [§2.8](#28-service-level-secrets-long-syntax). |
| `configs` | list \| map | Grant access to configs defined at top-level. See [§2.9](#29-service-level-configs-long-syntax). |

### Miscellaneous

| Key | Type | Description |
|-----|------|-------------|
| `platform` | string | Target platform for the service image (e.g., `linux/amd64`, `linux/arm64`). |
| `pull_policy` | string | When to pull the image: `always`, `never`, `missing` (default), `build`, `if_not_present`. |
| `profiles` | list | Assign service to named profiles. Service only starts when that profile is active. |
| `stdin_open` | bool | Keep stdin open (equivalent to `docker run -i`). |
| `tty` | bool | Allocate a pseudo-TTY (equivalent to `docker run -t`). |
| `scale` | int | Number of replicas (shorthand alternative to `deploy.replicas`). |
| `extends` | map | Inherit configuration from another service. See [§2.12](#212-extends). |
| `ulimits` | map | Set ulimits for the container. See [§2.10](#210-ulimits). |
| `blkio_config` | map | Block IO weight and device rate limits. See [§2.11](#211-blkio_config). |
| `runtime` | string | Specify container runtime (e.g., `nvidia` for GPU workloads). |
| `storage_opt` | map | Storage driver options (e.g., `size: 1G`). |
| `pids_limit` | int | Tune container PID limit. `-1` = unlimited. |
| `cgroup_parent` | string | Specify optional parent cgroup for the container. |
| `cgroup` | string | Cgroup namespace mode: `host` or `private`. |
| `isolation` | string | Container isolation technology: `default`, `process`, `hyperv` (Windows only). |
| `volumes_from` | list | Mount volumes from another container or service. |
| `post_start` | list | Lifecycle hooks run after container start (list of commands). |
| `pre_stop` | list | Lifecycle hooks run before container stop (list of commands). |

---

### 2.1 `build` Sub-Keys

```yaml
services:
  app:
    build:
      context: ./app
      dockerfile: Dockerfile.prod
      ...
```

| Key | Type | Description |
|-----|------|-------------|
| `context` | string | Path to build context directory or URL to a Git repo. |
| `dockerfile` | string | Path to Dockerfile relative to the context. |
| `dockerfile_inline` | string | Inline Dockerfile content as a string (alternative to a file). |
| `args` | list \| map | Build-time `ARG` values passed to the Dockerfile. |
| `cache_from` | list | Images to use as cache sources (e.g., registry images). |
| `cache_to` | list | Targets to export the build cache to. |
| `additional_contexts` | map | Extra named build contexts (e.g., for multi-stage or `FROM name` references). |
| `extra_hosts` | list | Add hosts to `/etc/hosts` during build (e.g., `"host.docker.internal:host-gateway"`). |
| `isolation` | string | Build isolation technology (primarily Windows). |
| `labels` | list \| map | Metadata labels to add to the built image. |
| `network` | string | Network mode for `RUN` instructions during build (e.g., `host`, `none`). |
| `no_cache` | bool | Disable build cache — always rebuild from scratch. |
| `pull` | bool | Force pulling the base image even if available locally. |
| `shm_size` | string | Size of shared memory (`/dev/shm`) during build. |
| `ssh` | list | SSH agent sockets/keys forwarded into the build (e.g., `default`). |
| `secrets` | list \| map | Secrets available during build (not baked into the image). |
| `tags` | list | Additional image tags to apply to the built image. |
| `target` | string | Multi-stage build target stage to stop at. |
| `platforms` | list | Target platforms for multi-arch builds (e.g., `linux/amd64`, `linux/arm64`). |
| `privileged` | bool | Allow privileged operations during build. |
| `ulimits` | map | Set ulimits during build. |
| `sbom` | bool \| string | Enable SBOM (Software Bill of Materials) attestation for the image. |
| `provenance` | bool \| string | Enable provenance attestation for the image. |

---

### 2.2 `deploy` Sub-Keys

> Primarily used with Docker Swarm. Some keys (like `resources`) are also honoured by `docker compose up`.

```yaml
services:
  app:
    deploy:
      replicas: 3
      ...
```

| Key | Type | Description |
|-----|------|-------------|
| `replicas` | int | Number of container instances to run. |
| `mode` | string | Deployment mode: `replicated` (default) or `global` (one per node). |
| `labels` | list \| map | Labels added to the service (not the container). |
| `update_config` | map | How the service is updated (rolling update config). |
| `rollback_config` | map | How the service is rolled back on failure. |
| `restart_policy` | map | Restart policy: `condition`, `delay`, `max_attempts`, `window`. |
| `resources` | map | Resource limits and reservations. |
| `placement` | map | Constraints and preferences for node placement. |
| `endpoint_mode` | string | Service endpoint mode: `vip` (default) or `dnsrr`. |

#### `deploy.resources`

```yaml
deploy:
  resources:
    limits:
      cpus: "0.50"
      memory: 512M
      pids: 100
    reservations:
      cpus: "0.25"
      memory: 128M
      devices:
        - capabilities: [gpu]
          driver: nvidia
          count: 1
```

| Key | Description |
|-----|-------------|
| `limits.cpus` | Maximum CPU shares (e.g., `"0.5"` = half a core). |
| `limits.memory` | Maximum memory (e.g., `512M`, `1G`). |
| `limits.pids` | Maximum number of processes. |
| `reservations.cpus` | Guaranteed CPU reservation. |
| `reservations.memory` | Guaranteed memory reservation. |
| `reservations.devices` | GPU/device reservations (capabilities, driver, count, device_ids). |

#### `deploy.update_config` / `deploy.rollback_config`

| Key | Description |
|-----|-------------|
| `parallelism` | Number of containers to update simultaneously. |
| `delay` | Delay between updates (e.g., `10s`). |
| `failure_action` | Action on failure: `pause`, `continue`, or `rollback`. |
| `monitor` | Duration to monitor after update (e.g., `60s`). |
| `max_failure_ratio` | Failure rate to tolerate before triggering `failure_action`. |
| `order` | `stop-first` or `start-first`. |

#### `deploy.restart_policy`

| Key | Description |
|-----|-------------|
| `condition` | `none`, `on-failure`, or `any`. |
| `delay` | Delay between restart attempts (e.g., `5s`). |
| `max_attempts` | Maximum restart attempts before giving up. |
| `window` | Time window to evaluate restart policy (e.g., `120s`). |

#### `deploy.placement`

```yaml
deploy:
  placement:
    constraints:
      - node.role == manager
      - node.labels.region == us-east
    preferences:
      - spread: node.labels.zone
    max_replicas_per_node: 2
```

| Key | Description |
|-----|-------------|
| `constraints` | List of placement constraint expressions. |
| `preferences` | Spread or pack replicas across node labels. |
| `max_replicas_per_node` | Cap replicas per node. |

---

### 2.3 `healthcheck` Sub-Keys

```yaml
healthcheck:
  test: ["CMD", "curl", "-f", "http://localhost/health"]
  interval: 30s
  timeout: 10s
  retries: 3
  start_period: 40s
  start_interval: 5s
  disable: false
```

| Key | Type | Description |
|-----|------|-------------|
| `test` | string \| list | Command to run. `["CMD", ...]`, `["CMD-SHELL", "..."]`, or `"NONE"` to disable. |
| `interval` | duration | Time between health checks (default: `30s`). |
| `timeout` | duration | Time to wait before marking check as failed (default: `30s`). |
| `retries` | int | Consecutive failures before marking container as unhealthy (default: `3`). |
| `start_period` | duration | Grace period before counting failures (container init time). |
| `start_interval` | duration | Interval between checks during `start_period`. |
| `disable` | bool | Set to `true` to disable the health check entirely. |

---

### 2.4 `logging` Sub-Keys

```yaml
logging:
  driver: json-file
  options:
    max-size: "10m"
    max-file: "3"
```

| Key | Type | Description |
|-----|------|-------------|
| `driver` | string | Log driver: `json-file` (default), `syslog`, `journald`, `gelf`, `fluentd`, `awslogs`, `splunk`, `none`, etc. |
| `options` | map | Driver-specific options (key-value string pairs). |

Common `json-file` options: `max-size`, `max-file`, `compress`, `labels`, `env`.  
Common `syslog` options: `syslog-address`, `syslog-facility`, `syslog-tag`.

---

### 2.5 Service-Level `volumes` Long Syntax

```yaml
volumes:
  # Short
  - ./data:/app/data
  - myvolume:/var/lib/data:ro

  # Long
  - type: bind
    source: ./static
    target: /opt/app/static
    read_only: true
    bind:
      create_host_path: true
      propagation: shared    # private|shared|slave|rprivate|rshared|rslave

  - type: volume
    source: mydata
    target: /data
    volume:
      nocopy: true
      subpath: subdir

  - type: tmpfs
    target: /tmp
    tmpfs:
      size: 100m
      mode: 0755

  - type: npipe             # Windows named pipes
    source: \\.\pipe\docker_engine
    target: \\.\pipe\docker_engine

  - type: cluster           # Cluster volumes (Swarm CSI)
    source: mycsivol
    target: /data
```

| Field | Description |
|-------|-------------|
| `type` | `volume`, `bind`, `tmpfs`, `npipe`, `image`, `cluster`. |
| `source` | Host path (bind), volume name, or named pipe. |
| `target` | Mount path inside the container. |
| `read_only` | Mount as read-only. |
| `bind.create_host_path` | Auto-create host directory if missing. |
| `bind.propagation` | Bind propagation mode. |
| `bind.selinux` | SELinux relabeling: `z` (shared) or `Z` (private). |
| `volume.nocopy` | Disable copying data from container to volume on mount. |
| `volume.subpath` | Mount a sub-directory of the volume. |
| `tmpfs.size` | Size limit for the tmpfs mount. |
| `tmpfs.mode` | File mode (octal) for the tmpfs mount. |
| `consistency` | `consistent`, `cached`, `delegated` (macOS legacy hint). |

---

### 2.6 Service-Level `networks` Long Syntax

```yaml
networks:
  frontend:
    aliases:
      - myapp
    ipv4_address: 172.16.0.10
    ipv6_address: "2001:db8::10"
    link_local_ips:
      - 169.254.0.1
    priority: 1000
    driver_opts:
      com.docker.network.driver.mtu: "1450"
    mac_address: "02:42:ac:11:00:02"
    gw_priority: 10
```

| Field | Description |
|-------|-------------|
| `aliases` | Alternative hostnames for this service on this network. |
| `ipv4_address` | Static IPv4 address on this network. |
| `ipv6_address` | Static IPv6 address on this network. |
| `link_local_ips` | List of link-local IPs to assign. |
| `priority` | Priority order when connecting to multiple networks (higher = first). |
| `driver_opts` | Driver-specific options for this service's network interface. |
| `mac_address` | MAC address assigned to the service's interface on this network. |
| `gw_priority` | Priority for selecting default gateway across networks. |

---

### 2.7 `depends_on` Long Syntax

```yaml
depends_on:
  db:
    condition: service_healthy
    restart: true
    required: true
  redis:
    condition: service_started
```

| Field | Values | Description |
|-------|--------|-------------|
| `condition` | `service_started`, `service_healthy`, `service_completed_successfully` | When the dependency is considered "ready". |
| `restart` | bool | Restart this service if the dependency restarts. |
| `required` | bool | If `false`, skip this dependency if it's not available. Default: `true`. |

---

### 2.8 Service-Level `secrets` Long Syntax

```yaml
secrets:
  # Short
  - my_secret

  # Long
  - source: my_secret
    target: /run/secrets/secret_key
    uid: "1000"
    gid: "1000"
    mode: 0440
```

| Field | Description |
|-------|-------------|
| `source` | Name of the secret as defined in the top-level `secrets` section. |
| `target` | Path inside the container (default: `/run/secrets/<source>`). |
| `uid` | User ID for the secret file. |
| `gid` | Group ID for the secret file. |
| `mode` | File permissions (octal, e.g., `0440`). |

---

### 2.9 Service-Level `configs` Long Syntax

```yaml
configs:
  # Short
  - my_config

  # Long
  - source: my_config
    target: /etc/app/config.yml
    uid: "1000"
    gid: "1000"
    mode: 0444
```

| Field | Description |
|-------|-------------|
| `source` | Name of the config as defined in the top-level `configs` section. |
| `target` | Mount path inside the container. |
| `uid` | User ID for the config file. |
| `gid` | Group ID for the config file. |
| `mode` | File permissions (octal). |

---

### 2.10 `ulimits`

```yaml
ulimits:
  nofile:
    soft: 65536
    hard: 65536
  nproc: 65535
  core: 0
  memlock:
    soft: -1
    hard: -1
```

Common ulimit names: `nofile` (open files), `nproc` (processes), `core` (core dump size), `memlock` (locked memory), `stack` (stack size), `cpu` (CPU time), `fsize` (file size).

---

### 2.11 `blkio_config`

```yaml
blkio_config:
  weight: 300
  weight_device:
    - path: /dev/sda
      weight: 400
  device_read_bps:
    - path: /dev/sdb
      rate: "12mb"
  device_write_bps:
    - path: /dev/sdb
      rate: "1024k"
  device_read_iops:
    - path: /dev/sdb
      rate: 120
  device_write_iops:
    - path: /dev/sdb
      rate: 30
```

| Key | Description |
|-----|-------------|
| `weight` | Relative block IO weight (10–1000, default 500). |
| `weight_device` | Per-device IO weight. Each entry: `path` + `weight`. |
| `device_read_bps` | Read rate limit in bytes/sec per device. Each entry: `path` + `rate`. |
| `device_write_bps` | Write rate limit in bytes/sec per device. |
| `device_read_iops` | Read IOPS limit per device. Each entry: `path` + `rate`. |
| `device_write_iops` | Write IOPS limit per device. |

---

### 2.12 `extends`

```yaml
services:
  web:
    extends:
      file: common.yaml     # optional — omit to use same file
      service: base-web
    ports:
      - "8080:80"
```

| Field | Description |
|-------|-------------|
| `file` | Path to the Compose file containing the base service. Omit to reference the current file. |
| `service` | Name of the service to extend from. |

> Inherited keys are merged. Scalars are overridden; lists and maps are merged. Circular references are not allowed.

---

## 3. Top-Level `networks`

```yaml
networks:
  frontend:
    driver: bridge
    ...
  backend:
    external: true
    name: my-pre-existing-net
```

| Key | Type | Description |
|-----|------|-------------|
| `driver` | string | Network driver: `bridge` (default), `host`, `overlay`, `ipvlan`, `macvlan`, `none`. |
| `driver_opts` | map | Driver-specific options passed to the network driver. |
| `attachable` | bool | Allow standalone containers to attach (Swarm overlay networks). |
| `enable_ipv6` | bool | Enable IPv6 networking. |
| `internal` | bool | Isolate the network from external connectivity. |
| `labels` | list \| map | Metadata labels on the network. |
| `external` | bool | Use a pre-existing network instead of creating one. |
| `name` | string | Custom name for the network (overrides the Compose-generated name). |
| `ipam` | map | IP Address Management configuration. |

#### `networks.<name>.ipam`

```yaml
networks:
  mynet:
    ipam:
      driver: default
      config:
        - subnet: 172.28.0.0/16
          ip_range: 172.28.5.0/24
          gateway: 172.28.5.254
          aux_addresses:
            myhost: 172.28.1.5
      options:
        foo: bar
```

| Field | Description |
|-------|-------------|
| `driver` | IPAM driver (`default` or custom plugin). |
| `config` | List of IPAM pool configs: `subnet`, `ip_range`, `gateway`, `aux_addresses`. |
| `options` | Driver-specific options map. |

---

## 4. Top-Level `volumes`

```yaml
volumes:
  mydata:                     # named volume, managed by Docker
    driver: local

  nfs-data:
    driver: local
    driver_opts:
      type: nfs
      o: addr=192.168.1.1,rw
      device: ":/path/on/server"

  external-vol:
    external: true
    name: my-existing-volume

  labeled-vol:
    labels:
      com.example.project: myapp
```

| Key | Type | Description |
|-----|------|-------------|
| `driver` | string | Volume driver: `local` (default) or custom plugin (e.g., `rexray/s3`). |
| `driver_opts` | map | Driver-specific options (e.g., NFS params, size hints). |
| `external` | bool | Use a pre-existing volume not managed by Compose. |
| `name` | string | Override the default volume name. |
| `labels` | list \| map | Metadata labels on the volume. |

---

## 5. Top-Level `secrets`

```yaml
secrets:
  db_password:
    file: ./secrets/db_password.txt   # content from file

  api_key:
    environment: MY_API_KEY           # content from host env var

  preexisting:
    external: true
    name: my-docker-secret

  inline_secret:
    content: "supersecretvalue"       # inline literal value
```

| Key | Type | Description |
|-----|------|-------------|
| `file` | string | Path to a file whose contents become the secret value. |
| `environment` | string | Name of a host environment variable whose value becomes the secret. |
| `content` | string | Literal string value for the secret (inline). |
| `external` | bool | Use a pre-existing Docker secret (Swarm). |
| `name` | string | Override the name of the external secret. |
| `labels` | list \| map | Metadata labels. |
| `driver` | string | Secret driver name (custom). |
| `driver_opts` | map | Options passed to the secret driver. |
| `template_driver` | string | Template driver used to render the secret (e.g., `golang`). |

---

## 6. Top-Level `configs`

```yaml
configs:
  app_config:
    file: ./config/app.conf

  remote_config:
    external: true
    name: prod-app-config

  inline_config:
    content: |
      key=value
      setting=true
```

| Key | Type | Description |
|-----|------|-------------|
| `file` | string | Path to a file whose contents become the config value. |
| `content` | string | Inline literal config content. |
| `environment` | string | Host environment variable to use as config content. |
| `external` | bool | Use a pre-existing Docker config (Swarm). |
| `name` | string | Override the name of the external config. |
| `labels` | list \| map | Metadata labels. |
| `template_driver` | string | Template engine for the config content. |

---

## 7. `include`

Allows splitting a large Compose setup into multiple files.

```yaml
include:
  - ./infra/db.yaml
  - path: ./infra/cache.yaml
    env_file: ./infra/.env
    project_directory: ./infra
```

| Field | Type | Description |
|-------|------|-------------|
| `path` | string \| list | Path(s) to the included Compose file(s). |
| `env_file` | string \| list | `.env` file(s) to use when processing the included file. |
| `project_directory` | string | Base directory for relative paths in the included file. |

---

## 8. Variable Interpolation & `.env`

Compose supports `${VAR}` substitution in most string values.

| Syntax | Behaviour |
|--------|-----------|
| `${VAR}` | Substitutes the value of `VAR`. Empty string if unset. |
| `${VAR:-default}` | Uses `default` if `VAR` is unset **or** empty. |
| `${VAR-default}` | Uses `default` only if `VAR` is unset (not if empty). |
| `${VAR:?error msg}` | Fails with `error msg` if `VAR` is unset or empty. |
| `${VAR?error msg}` | Fails with `error msg` if `VAR` is unset. |
| `${VAR:+replacement}` | Uses `replacement` if `VAR` is set and non-empty. |
| `$$` | Literal `$` sign (escapes interpolation). |

The `.env` file at the project root is loaded automatically. Additional env files can be specified via `--env-file` on the CLI.

---

## 9. YAML Anchors & Fragments

Compose files are valid YAML. You can use native YAML anchors to avoid repetition:

```yaml
x-common-env: &common-env
  RAILS_ENV: production
  LOG_LEVEL: info

x-deploy-defaults: &deploy-defaults
  deploy:
    replicas: 2
    resources:
      limits:
        memory: 512M

services:
  web:
    <<: *deploy-defaults
    environment:
      <<: *common-env
      APP_ROLE: web

  worker:
    <<: *deploy-defaults
    environment:
      <<: *common-env
      APP_ROLE: worker
```

> Keys starting with `x-` are **extension fields** — they are ignored by Compose but useful as YAML anchors/fragments.

---

## 10. Watch Mode (Compose Watch)

Enables file-watching for live development without volume bind mounts.

```yaml
services:
  app:
    develop:
      watch:
        - action: sync
          path: ./src
          target: /app/src
          ignore:
            - node_modules/

        - action: rebuild
          path: ./Dockerfile

        - action: sync+restart
          path: ./config
          target: /app/config
```

| Field | Description |
|-------|-------------|
| `action` | `sync` (copy changes), `rebuild` (rebuild image), `sync+restart` (sync then restart). |
| `path` | Host path to watch. |
| `target` | Container path to sync into (for `sync` / `sync+restart`). |
| `ignore` | List of patterns to exclude from watching. |

---

## 11. Full Annotated Example

```yaml
# -----------------------------------------------
# PROJECT
# -----------------------------------------------
name: my-fullstack-app

# -----------------------------------------------
# EXTENSION FIELDS (YAML anchors)
# -----------------------------------------------
x-common-labels: &common-labels
  maintainer: "devops@example.com"
  project: "myapp"

x-logging: &default-logging
  driver: json-file
  options:
    max-size: "10m"
    max-file: "5"

# -----------------------------------------------
# SERVICES
# -----------------------------------------------
services:

  # --- Frontend ---
  frontend:
    build:
      context: ./frontend
      dockerfile: Dockerfile
      target: production
      args:
        NODE_ENV: production
      cache_from:
        - myregistry.io/frontend:cache
      labels: *common-labels
    image: myregistry.io/frontend:latest
    container_name: frontend
    hostname: frontend
    restart: unless-stopped
    ports:
      - target: 80
        published: "3000"
        protocol: tcp
    networks:
      - public
    depends_on:
      api:
        condition: service_healthy
    logging: *default-logging
    labels: *common-labels
    profiles:
      - full
    develop:
      watch:
        - action: sync
          path: ./frontend/src
          target: /app/src

  # --- API ---
  api:
    build:
      context: ./api
      secrets:
        - npmrc
    image: myregistry.io/api:latest
    restart: unless-stopped
    environment:
      - NODE_ENV=production
      - DB_HOST=db
    env_file:
      - .env
    ports:
      - "4000:4000"
    volumes:
      - type: bind
        source: ./api/config
        target: /app/config
        read_only: true
    networks:
      - public
      - internal
    depends_on:
      db:
        condition: service_healthy
      redis:
        condition: service_started
    healthcheck:
      test: ["CMD", "wget", "-qO-", "http://localhost:4000/health"]
      interval: 30s
      timeout: 5s
      retries: 3
      start_period: 10s
    secrets:
      - source: db_password
        target: /run/secrets/db_pass
        mode: 0400
    configs:
      - source: app_config
        target: /app/config/app.conf
    deploy:
      replicas: 2
      resources:
        limits:
          cpus: "0.5"
          memory: 256M
        reservations:
          memory: 128M
      restart_policy:
        condition: on-failure
        max_attempts: 3
    logging: *default-logging
    labels: *common-labels

  # --- Database ---
  db:
    image: postgres:16-alpine
    restart: unless-stopped
    environment:
      POSTGRES_USER: app
      POSTGRES_DB: myapp
      POSTGRES_PASSWORD_FILE: /run/secrets/db_pass
    volumes:
      - type: volume
        source: pgdata
        target: /var/lib/postgresql/data
    networks:
      - internal
    secrets:
      - db_password
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U app"]
      interval: 10s
      timeout: 5s
      retries: 5
    deploy:
      resources:
        limits:
          memory: 512M

  # --- Cache ---
  redis:
    image: redis:7-alpine
    restart: unless-stopped
    command: ["redis-server", "--maxmemory", "128mb", "--maxmemory-policy", "allkeys-lru"]
    volumes:
      - redisdata:/data
    networks:
      - internal
    healthcheck:
      test: ["CMD", "redis-cli", "ping"]
      interval: 10s
      timeout: 3s
      retries: 3

  # --- Worker ---
  worker:
    extends:
      service: api
    command: ["node", "worker.js"]
    ports: []               # clear inherited ports
    deploy:
      replicas: 1
    profiles:
      - full

# -----------------------------------------------
# NETWORKS
# -----------------------------------------------
networks:
  public:
    driver: bridge
    labels: *common-labels

  internal:
    driver: bridge
    internal: true
    ipam:
      config:
        - subnet: 10.10.0.0/24
          gateway: 10.10.0.1

# -----------------------------------------------
# VOLUMES
# -----------------------------------------------
volumes:
  pgdata:
    driver: local
    labels: *common-labels

  redisdata:
    driver: local

# -----------------------------------------------
# SECRETS
# -----------------------------------------------
secrets:
  db_password:
    file: ./secrets/db_password.txt

  npmrc:
    environment: NPM_TOKEN

# -----------------------------------------------
# CONFIGS
# -----------------------------------------------
configs:
  app_config:
    file: ./config/app.conf
```

---

## Quick Reference Card

```
compose.yaml
├── name
├── include[]
│   ├── path
│   ├── env_file
│   └── project_directory
├── services
│   └── <name>
│       ├── image / build{context,dockerfile,args,target,cache_from,...}
│       ├── command / entrypoint
│       ├── environment / env_file
│       ├── ports[] / expose[]
│       ├── volumes[] / volumes_from[] / tmpfs
│       ├── networks{} / network_mode
│       ├── depends_on{condition,restart,required}
│       ├── healthcheck{test,interval,timeout,retries,start_period}
│       ├── deploy{replicas,resources,restart_policy,update_config,...}
│       ├── restart / stop_signal / stop_grace_period
│       ├── secrets[] / configs[]
│       ├── logging{driver,options}
│       ├── labels / annotations
│       ├── user / working_dir / hostname / domainname
│       ├── stdin_open / tty / init / read_only
│       ├── cap_add / cap_drop / privileged / security_opt / sysctls
│       ├── devices / device_cgroup_rules
│       ├── ulimits{} / blkio_config{} / shm_size
│       ├── platform / pull_policy / profiles[]
│       ├── extends{file,service}
│       ├── pids_limit / cgroup / cgroup_parent
│       ├── runtime / storage_opt
│       ├── extra_hosts / dns / dns_search / dns_opt
│       ├── links / external_links
│       ├── post_start[] / pre_stop[]
│       └── develop{watch[{action,path,target,ignore}]}
├── networks
│   └── <name>
│       ├── driver / driver_opts
│       ├── ipam{driver,config[{subnet,gateway,ip_range}]}
│       ├── internal / attachable / enable_ipv6
│       ├── external / name
│       └── labels
├── volumes
│   └── <name>
│       ├── driver / driver_opts
│       ├── external / name / labels
├── secrets
│   └── <name>
│       ├── file / environment / content
│       ├── external / name
│       └── driver / driver_opts / template_driver
└── configs
    └── <name>
        ├── file / environment / content
        ├── external / name
        └── template_driver
```

---

*Reference: [docs.docker.com/reference/compose-file](https://docs.docker.com/reference/compose-file/) — Compose Specification (2024–2025)*