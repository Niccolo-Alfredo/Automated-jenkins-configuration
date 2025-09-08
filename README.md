# Jenkins Automation con Vagrant e Ansible

Questo progetto automatizza completamente la configurazione di un ambiente Jenkins con master e agenti utilizzando **Vagrant**, **Ansible** e **Docker**. Il setup include l'installazione automatica di Jenkins, la configurazione di sicurezza, l'installazione di plugin essenziali e la creazione di agenti Docker tramite REST API.

## Caratteristiche Principali

- **Setup completo automatizzato** di Jenkins Master in container Docker
- **Configurazione di sicurezza** con utente admin predefinito
- **Installazione automatica** di plugin essenziali
- **Creazione multipla di agenti Jenkins** tramite REST API
- **Rete Docker dedicata** con IP statici per master e agenti
- **Agenti con supporto Docker-in-Docker** per pipeline CI/CD avanzate
- **Configurazione idempotente** con Ansible

## Prerequisiti

- **Vagrant** 2.2+
- **VirtualBox** o **libvirt** (KVM)
- **Git**

Il progetto è configurato per **libvirt/KVM** su sistemi Linux, ma può essere facilmente adattato per VirtualBox.

## Architettura

```
┌─────────────────────────────────────────┐
│              VM Jenkins                 │
│  (Rocky Linux 9 - 192.168.56.10)      │
│                                         │
│  ┌─────────────────────────────────────┐│
│  │        Docker Network               ││
│  │      (jenkins-net)                  ││
│  │                                     ││
│  │  ┌─────────────┐  ┌───────────────┐ ││
│  │  │Jenkins      │  │Jenkins Agent-1│ ││
│  │  │Master       │  │172.25.0.11    │ ││
│  │  │172.25.0.10  │  │               │ ││
│  │  │:8080, :50000│  │               │ ││
│  │  └─────────────┘  └───────────────┘ ││
│  │                                     ││
│  │  ┌───────────────┐  ┌──────────────┐││
│  │  │Jenkins Agent-2│  │Jenkins       │││
│  │  │172.25.0.12    │  │Agent-N...    │││
│  │  │               │  │              │││
│  │  └───────────────┘  └──────────────┘││
│  └─────────────────────────────────────┘│
└─────────────────────────────────────────┘
```

## Struttura del Progetto

```
jenkins-automation/
├── README.md
├── Vagrantfile                     # Configurazione VM                     
├── main.yml                        # Playbook principale
├── .gitignore 
└── ansible/
    ├── ansible.cfg                     # Configurazione Ansible
    ├── inventory.ini                   # Inventory degli host
    └── yaml/
        ├── docker_setup.yml        # Setup Docker e rete
        └── master/
        │   ├── jenkins_master.yml  # Setup Jenkins Master
        │   ├── cleanup.yml         # Pulizia ambiente
        │   ├── wait_for_jenkins.yml # Attesa avvio Jenkins
        │   └── templates/
        │       ├── init-config.groovy.j2    # Config sicurezza
        │       └── init-plugins.groovy.j2   # Installazione plugin
        └── agent/
            ├── jenkins_agent_multi.yml      # Creazione agenti multipli
            ├── jenkins_agent_single.yml     # Task per singolo agente
            └── templates/
                ├── jenkins_agent_create.groovy.j2  # Script creazione agente
                └── jenkins-agent.Dockerfile.j2     # Dockerfile agente personalizzato
```

## Installazione e Utilizzo

### 1. Clona il Repository

```bash
git clone <repository-url>
cd jenkins-automation
```

### 2. Avvia l'Ambiente

```bash
# Avvia e provisiona la VM
vagrant up

# In caso di errori, riprova il provisioning
vagrant provision
```

### 3. Accesso a Jenkins

Una volta completato il setup:

- **URL**: http://192.168.56.10:8080
- **Username**: `admin`
- **Password**: `password`

### 4. Personalizzazione Agenti

Per modificare il numero di agenti, modifica il file `ansible/yaml/agent/jenkins_agent_multi.yml`:

```yaml
vars:
  agent_count: 3  # Cambia questo numero per creare più agenti
```

Quindi riavvia il provisioning:

```bash
vagrant provision
```

## Configurazione

### Variabili Principali

**Jenkins Master (`jenkins_master.yml`)**:
```yaml
jenkins_master_url: "http://192.168.56.10:8080"
jenkins_admin_user: "admin" 
jenkins_admin_password: "password"
```

**Agenti (`jenkins_agent_multi.yml`)**:
```yaml
agent_count: 2                    # Numero di agenti da creare
agent_base_name: "agent"          # Nome base degli agenti
agent_executors: 1                # Esecutori per agente
agent_label: "linux docker"       # Label degli agenti
```

**Rete Docker (`docker_setup.yml`)**:
```yaml
docker_network_name: jenkins-net
docker_subnet: 172.25.0.0/16
jenkins_master_ip: 172.25.0.10
```

## Plugin Installati Automaticamente

- **git**: Supporto repository Git
- **workflow-aggregator**: Pipeline Jenkins
- **configuration-as-code**: Configuration as Code
- **matrix-auth**: Autorizzazioni matriciali
- **credentials-binding**: Binding credenziali
- **build-timeout**: Timeout build
- **timestamper**: Timestamp nei log

## Sicurezza

- **Utente admin** configurato automaticamente
- **Protezione CSRF** abilitata
- **Accesso anonimo** disabilitato
- **CLI Jenkins** disabilitata per sicurezza
- **Esecutori master** impostati a 0 (best practice)

## Docker-in-Docker

Ogni agente Jenkins è configurato con:
- **Docker CLI** installato
- **Accesso al socket Docker** dell'host
- **Gruppi e permessi** configurati automaticamente
- **Supporto completo** per pipeline Docker

## Comandi Utili

```bash
# Riavvia completamente l'ambiente
vagrant destroy -f && vagrant up

# Solo reprovisioning
vagrant provision

# SSH nella VM
vagrant ssh jenkins

# Verifica stato container
vagrant ssh jenkins -c "docker ps"

# Log Jenkins Master
vagrant ssh jenkins -c "docker logs jenkins-master"

# Log agenti
vagrant ssh jenkins -c "docker logs jenkins-agent-1"
```

## Troubleshooting

### Jenkins non si avvia
```bash
# Verifica i log del container
vagrant ssh jenkins -c "docker logs jenkins-master"

# Riavvia il container
vagrant ssh jenkins -c "docker restart jenkins-master"
```

### Agenti offline
```bash
# Verifica la connettività di rete
vagrant ssh jenkins -c "docker network ls"
vagrant ssh jenkins -c "docker network inspect jenkins-net"

# Riavvia gli agenti
vagrant ssh jenkins -c "docker restart jenkins-agent-1"
```

### Permessi Docker
```bash
# Verifica i permessi del socket Docker
vagrant ssh jenkins -c "ls -la /var/run/docker.sock"

# Test Docker negli agenti
vagrant ssh jenkins -c "docker exec jenkins-agent-1 docker --version"
```