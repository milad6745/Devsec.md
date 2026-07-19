No, you do not need one global/general Vault Agent configuration for the entire Vault deployment.

There are three distinct components here:

1. Vault server configuration
Configures the Vault server pods, storage, listeners, HA, TLS, etc.


2. Vault Agent Injector configuration
Configures the Kubernetes admission webhook itself.


3. Application-specific Vault Agent configuration
Configures the Vault Agent sidecar injected into a particular application pod: authentication method, RoleID/SecretID paths, templates, sinks, and rendered files.



Your ConfigMap in the image is category 3. That approach is valid. The injector mounts the ConfigMap into the injected Vault Agent containers and starts Vault Agent with that configuration. HashiCorp supports application-specific Agent ConfigMaps through the vault.hashicorp.com/agent-configmap annotation. 

However, for your external Docker host, the Kubernetes injector does not participate. The injector only mutates Kubernetes pods. Therefore, the external host must run its own Vault Agent process/container and receive its own AppRole-specific configuration.

Important distinction

Your previous application used:

method {
  type = "kubernetes"

  config = {
    role = "myapp-role"
  }
}

That works because the injected Agent can use the Kubernetes service-account JWT mounted in the pod.

For AppRole, replace that authentication configuration with:

method {
  type = "approle"

  config = {
    role_id_file_path   = "/vault/auth/role_id"
    secret_id_file_path = "/vault/auth/secret_id"
  }
}

Vault Agent reads the RoleID and SecretID from files. By default, it removes the SecretID file after reading it and caches the credential value internally for subsequent authentication attempts. 

Scenario A: AppRole for a Kubernetes application

You can use almost the same ConfigMap approach shown in your image.

1. Application-specific ConfigMap

apiVersion: v1
kind: ConfigMap
metadata:
  name: myapp-approle-vault-agent-config
  namespace: nasiri
data:
  template.hcl: |
    {{- with secret "secret/data/myapp/database" -}}
    export DB_USERNAME="{{ .Data.data.username }}"
    export DB_PASSWORD="{{ .Data.data.password }}"
    {{- end }}

  config-init.hcl: |
    exit_after_auth = true
    pid_file = "/vault/secrets/.pid"

    auto_auth {
      method "approle" {
        config = {
          role_id_file_path   = "/vault/auth/role_id"
          secret_id_file_path = "/vault/auth/secret_id"
        }
      }

      sink "file" {
        config = {
          path = "/vault/secrets/token"
          mode = 0600
        }
      }
    }

    template {
      source      = "/vault/configs/template.hcl"
      destination = "/vault/secrets/config"
      perms       = 0600
    }

    vault {
      address = "https://vault-active.nasiri.svc.cluster.local:32023"
      ca_cert = "/vault/tls/ca.crt"
    }

  config.hcl: |
    exit_after_auth = false
    pid_file = "/vault/secrets/.pid"

    auto_auth {
      method "approle" {
        config = {
          role_id_file_path   = "/vault/auth/role_id"
          secret_id_file_path = "/vault/auth/secret_id"
        }
      }

      sink "file" {
        config = {
          path = "/vault/secrets/token"
          mode = 0600
        }
      }
    }

    template {
      source      = "/vault/configs/template.hcl"
      destination = "/vault/secrets/config"
      perms       = 0600
    }

    vault {
      address = "https://vault-active.nasiri.svc.cluster.local:32023"
      ca_cert = "/vault/tls/ca.crt"
    }

The injector expects:

config-init.hcl for the init Agent, with exit_after_auth = true

config.hcl for the sidecar Agent, with exit_after_auth = false


It mounts these under /vault/configs. 

2. Store RoleID separately

RoleID is not normally treated like a high-value secret, but it should still not be embedded directly in the ConfigMap.

apiVersion: v1
kind: Secret
metadata:
  name: myapp-approle-role-id
  namespace: nasiri
type: Opaque
stringData:
  role_id: "YOUR_ROLE_ID"

Do not place a reusable SecretID in a Kubernetes Secret when your objective is a one-use, short-TTL SecretID.

3. Mount AppRole credential files

The application Deployment would need volumes for:

RoleID

short-lived SecretID

Vault CA

Agent configuration


Conceptually:

apiVersion: apps/v1
kind: Deployment
metadata:
  name: myapp
  namespace: nasiri
spec:
  replicas: 1
  selector:
    matchLabels:
      app: myapp
  template:
    metadata:
      labels:
        app: myapp
      annotations:
        vault.hashicorp.com/agent-inject: "true"
        vault.hashicorp.com/agent-configmap: "myapp-approle-vault-agent-config"
    spec:
      containers:
        - name: myapp
          image: your-registry/myapp:latest

          volumeMounts:
            - name: approle-role-id
              mountPath: /vault/auth/role_id
              subPath: role_id
              readOnly: true

            - name: approle-secret-id
              mountPath: /vault/auth/secret_id
              subPath: secret_id
              readOnly: true

            - name: vault-ca
              mountPath: /vault/tls
              readOnly: true

      volumes:
        - name: approle-role-id
          secret:
            secretName: myapp-approle-role-id

        - name: approle-secret-id
          secret:
            secretName: myapp-bootstrap-secret-id

        - name: vault-ca
          secret:
            secretName: vault-ca

There is an operational problem with this design: if the SecretID has:

secret_id_num_uses = 1

the init Agent may consume it first, leaving the sidecar unable to authenticate separately.

Even if Agent state or cached credentials make a particular startup work, a pod restart can fail because the same SecretID is already consumed.

Therefore, for Kubernetes pods, Kubernetes auth is usually the correct method. AppRole adds a bootstrap credential-distribution problem that Kubernetes auth already solves using the pod service-account identity.

Recommended split for your environment

Use:

Kubernetes applications → Kubernetes auth
External Docker/bare-metal applications → AppRole auth

That is the cleanest design.

Your existing Kubernetes ConfigMap should therefore remain Kubernetes-auth based:

auto_auth {
  method "kubernetes" {
    config = {
      role = "myapp-role"
    }
  }
}

For the external host 192.168.72.43, create a separate application-specific AppRole Agent configuration.

Scenario B: AppRole for the external Docker application

The Kubernetes injector cannot inject into this Docker container. You reproduce the same architectural pattern with Docker Compose:

ConfigMap equivalent       → host-mounted config directory
Kubernetes Secret          → protected host file or tmpfs
Injected sidecar           → Vault Agent container
emptyDir /vault/secrets    → shared Docker volume
Application pod container  → application container

Directory layout on 192.168.72.43

/opt/myapp/
├── docker-compose.yml
├── vault-agent/
│   ├── config.hcl
│   └── templates/
│       └── app.env.ctmpl
├── vault/
│   ├── ca.crt
│   └── role_id
└── bootstrap/
    └── fetch-secret-id.sh

/run/myapp-vault/
└── secret_id

/run should be backed by tmpfs on normal Linux systems.

/opt/myapp/vault-agent/config.hcl

exit_after_auth = false
pid_file = "/vault/runtime/agent.pid"

vault {
  address = "https://192.168.73.128:32023"
  ca_cert = "/vault/tls/ca.crt"
}

auto_auth {
  method "approle" {
    mount_path = "auth/approle"

    config = {
      role_id_file_path   = "/vault/auth/role_id"
      secret_id_file_path = "/vault/runtime/secret_id"
    }
  }

  sink "file" {
    config = {
      path = "/vault/runtime/token"
      mode = 0600
    }
  }
}

template {
  source      = "/vault/templates/app.env.ctmpl"
  destination = "/vault/runtime/app.env"
  perms       = 0600

  error_on_missing_key = true
}

Template

/opt/myapp/vault-agent/templates/app.env.ctmpl:

{{- with secret "secret/data/myapp/database" }}
DB_USERNAME={{ .Data.data.username }}
DB_PASSWORD={{ .Data.data.password }}
{{- end }}

Vault Agent templates can render KV secrets, dynamic credentials and certificates into application-specific files. 

Docker Compose

services:
  vault-agent:
    image: hashicorp/vault:1.21
    container_name: myapp-vault-agent
    restart: unless-stopped

    command:
      - vault
      - agent
      - -config=/vault/config/config.hcl

    volumes:
      - ./vault-agent/config.hcl:/vault/config/config.hcl:ro
      - ./vault-agent/templates:/vault/templates:ro
      - ./vault/role_id:/vault/auth/role_id:ro
      - ./vault/ca.crt:/vault/tls/ca.crt:ro
      - vault-runtime:/vault/runtime

    cap_drop:
      - ALL

    security_opt:
      - no-new-privileges:true

  myapp:
    image: your-registry/myapp:latest
    container_name: myapp
    restart: unless-stopped
    depends_on:
      - vault-agent

    volumes:
      - vault-runtime:/vault/secrets:ro

    entrypoint:
      - /bin/sh
      - -ec
      - |
        until [ -s /vault/secrets/app.env ]; do
          echo "Waiting for Vault Agent..."
          sleep 1
        done

        set -a
        . /vault/secrets/app.env
        set +a

        exec /app/start

volumes:
  vault-runtime:
    driver_opts:
      type: tmpfs
      device: tmpfs
      o: size=16m,mode=0700

This is the Docker equivalent of your Kubernetes sidecar model.

Where the wrapped SecretID fits

The Agent does not normally request its own SecretID. Doing so would require giving the external host a provisioning token that can generate SecretIDs, which defeats the bootstrap security model.

The secure flow is:

SecretID provisioner
        |
        | generate wrapped SecretID
        v
External host bootstrap process
        |
        | unwrap once
        v
/run/.../secret_id
        |
        | Vault Agent reads and deletes it
        v
AppRole login → renewable Vault token

The SecretID provisioner can be:

GitLab CI/CD

a central provisioning service

an orchestrator

a configuration-management controller

a restricted bootstrap service


It should possess a narrowly scoped token capable only of generating SecretIDs for the specific AppRole.

The external host should not possess that provisioning token permanently.

The key correction to the previous scenario

This operation is unsafe when performed anonymously from the external host:

curl -X POST \
  "$VAULT_ADDR/v1/auth/approle/role/myapp/secret-id"

That endpoint requires an authenticated Vault token with update permission on:

auth/approle/role/myapp/secret-id

Therefore, the external host cannot “fetch a wrapped SecretID unattended” unless one of the following is true:

1. it already holds another trusted machine credential;


2. a CI/CD or provisioning service delivers the wrapped token;


3. it uses a cloud/platform identity for initial authentication;


4. it stores a long-lived provisioning credential locally, which is not recommended.



Wrapping protects delivery of the SecretID; it does not solve the question of who is authorized to generate it.

Final recommendation

For your exact topology:

Vault injector pod:
    one shared admission controller
    no application AppRole credentials
    no global application Agent config

Kubernetes consumer:
    application-specific ConfigMap
    Kubernetes auth
    service-account-bound Vault role

External Docker consumer:
    separate Vault Agent container
    application-specific config.hcl
    AppRole
    RoleID mounted read-only
    one-use SecretID delivered by GitLab/provisioner
    rendered secrets stored in a tmpfs shared volume

So the answer is:

> You need a Vault Agent configuration for every Agent instance, but you do not need one global configuration in the Vault Helm chart. In Kubernetes, the injector generates or mounts the application-specific configuration. Outside Kubernetes, you must supply the application-specific config.hcl to the Vault Agent container or host service yourself.