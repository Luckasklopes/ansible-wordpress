# Ansible WordPress Automation

Automação de gerenciamento de múltiplos ambientes WordPress containerizados utilizando Ansible, Docker Compose e WP-CLI.

## Objetivo

Centralizar e automatizar tarefas repetitivas de administração de sites WordPress executados em containers Docker.

O projeto foi desenvolvido inicialmente para automatizar a verificação de atualizações de plugins em múltiplos sites, utilizando um container compartilhado de WP-CLI configurado dinamicamente pelo Ansible.

## Arquitetura

```text
                    Ansible Controller
                           │
                           │ SSH
                           ▼
                      Docker Host
                           │
             ┌─────────────┴─────────────┐
             │                           │
        Site A                      Site B
             │                           │
        Docker Compose             Docker Compose
             │                           │
        WordPress + DB             WordPress + DB
             ▲                           ▲
             │                           │
             └────────── WP-CLI ─────────┘
```

O Ansible atua como controlador central, conectando-se aos hosts gerenciados por SSH.

Para operações WordPress, um container `wordpress:cli` é configurado dinamicamente para acessar o volume, a rede Docker e o arquivo `.env` correspondente ao site que está sendo processado.

## Estrutura

```text
ansible/
├── .gitignore
├── README.md
├── docker-compose.yml
├── inventory.ini
│
├── group_vars/
│   ├── docker_sites.yml
│   └── docker_sites.example.yml
│
├── playbooks/
│   ├── wpcli.yml
│   └── check_plugins.yml
│
├── templates/
│   └── wpcli-compose.yml.j2
│
└── ssh/
    └── .gitkeep
```

## Componentes

### Ansible

Responsável pela orquestração das tarefas e gerenciamento dos hosts.

### Docker

Os sites WordPress são executados individualmente através de Docker Compose.

### WP-CLI

Um container temporário `wordpress:cli` é utilizado para executar operações administrativas nos sites.

O container recebe dinamicamente:

* volume do WordPress;
* rede Docker;
* arquivo `.env`;
* configuração do banco de dados.

### Jinja2

Templates Jinja2 são utilizados para gerar dinamicamente a configuração do container WP-CLI.

## Funcionalidades atuais

* Conexão remota via SSH utilizando usuário dedicado de automação.
* Inventário de hosts Ansible.
* Gerenciamento de múltiplos sites WordPress.
* Geração dinâmica do Compose do WP-CLI.
* Verificação de atualizações disponíveis de plugins.
* Execução do WP-CLI dentro de containers temporários.
* Coleta dos resultados das execuções pelo Ansible.
* Configuração individual de volume, rede e `.env` por site.

## Configuração

Os dados específicos da infraestrutura são mantidos em:

```text
group_vars/docker_sites.yml
```

Um exemplo da estrutura esperada pode ser encontrado em:

```text
group_vars/docker_sites.example.yml
```

Dados sensíveis e configurações específicas de ambiente não devem ser versionados.

## Execução

Para verificar atualizações de plugins:

```bash
ansible-playbook -i inventory.ini playbooks/wpcli.yml
```

O playbook percorre os sites definidos em `wp_sites` e executa a consulta de atualizações individualmente.

## Roadmap

* [ ] Melhorar tratamento e apresentação dos resultados.
* [ ] Detectar automaticamente ausência de atualizações.
* [ ] Automatizar atualização de plugins.
* [ ] Verificar versão do WordPress Core.
* [ ] Verificar versões de PHP e MariaDB/MySQL.
* [ ] Criar automação para provisionamento de novos sites.
* [ ] Gerar credenciais automaticamente durante o provisionamento.
* [ ] Integrar notificações.
* [ ] Integrar métricas e dashboards.
* [ ] Integrar Semaphore UI.
* [ ] Avaliar integração com Grafana.
* [ ] Implementar gerenciamento de segredos com Ansible Vault.

## Segurança

Credenciais, arquivos `.env`, chaves SSH e demais informações sensíveis não fazem parte do repositório.

O projeto utiliza arquivos `.example` para documentar a estrutura necessária sem expor dados da infraestrutura real.
