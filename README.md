# Zabbix Monitoring Template for IBM DB2

![Zabbix](https://img.shields.io/badge/Zabbix-6.0%2B-red)
![DB2](https://img.shields.io/badge/IBM%20DB2-Monitoring-blue)
![License](https://img.shields.io/badge/License-MIT-green)

Este repositório contém um template personalizado para monitoramento de instâncias **IBM DB2** via Zabbix. 

**Diferencial:** Este projeto nasceu da necessidade real de monitorar o DB2 em ambientes de produção, dado que não existem templates nativos robustos na comunidade Zabbix.

## Funcionalidades Monitoradas

O template utiliza `UserParameters` para coletar métricas essenciais diretamente do banco de dados:

* **Disponibilidade:** Status da instância (Up/Down) e tempo de resposta de query.
* **Performance:** Uso de CPU e Memória RAM exclusivo dos processos DB2.
* **Armazenamento:** Tamanho total do banco (GB), capacidade e uso de logs de transação.
* **Conexões:** Contagem de usuários ativos e status de conexões.
* **Saúde & Backup:** Status do último backup e contagem de Slow Queries (> 5s).
* **Licenciamento:** Nome do produto, expiração e tipo de licença.

## Configuração e Instalação
### 1. Pré-requisitos
O usuário do Zabbix precisa de permissão para executar comandos como o usuário da instância DB2 (ex: `db2ecp`). Adicione ao seu arquivo `sudoers`:

### 2. Configuração dos UserParameters
Para que o Zabbix colete os dados, você deve criar um arquivo de configuração no diretório do Zabbix Agent (geralmente `/etc/zabbix/zabbix_agentd.d/db2_monitor.conf`).

### Observação Técnica Importante:

Como administrador altere o shell do usuário ou use o parâmetro `-s /bin/bash` no comando `su` para garantir a compatibilidade:
`sudo su - db2ecp -s /bin/bash -c "comando_db2"`

Abaixo estão os parâmetros de configuração utilizados neste template. **Atenção:** Substitua `db2ecp` pelo usuário da sua instância (ex: `db2inst1`).

```bash
### --- Monitoramento Geral ---
# Status da instância (1=Up, 0=Down)
UserParameter=db2.status, pgrep db2sysc >/dev/null && echo 1 || echo 0
# Tempo de resposta de query simples
UserParameter=db2.query_time, sudo su - db2ecp -c "db2 -x 'SELECT 1 FROM SYSIBM.SYSDUMMY1;'"
# Quantidade de usuários conectados
UserParameter=db2.count_users, sudo su - db2ecp -c "db2 -x 'SELECT COUNT(*) FROM SYSIBM.SYSDBAUTH;'"

### --- Recursos de Hardware ---
# Uso de CPU pelo DB2 (%)
UserParameter=db2.cpu_usage, ps -C db2sysc -o %cpu --no-headers | awk '{s+=$1} END {print s}'
# Uso de memória pelo DB2 (MB)
UserParameter=db2.memory_usage, ps -C db2sysc -o rss --no-headers | awk '{s+=$1} END {print s}'

### --- Armazenamento e Logs ---
# Tamanho do banco de dados (GB)
UserParameter=db2.db_size, sudo su - db2ecp -c "db2 -x 'SELECT SUM(TBSP_USED_PAGES * 4096) / 1024 AS TOTAL_SIZE_GB FROM SYSIBMADM.TBSP_UTILIZATION;'"
# Porcentagem de uso do log de transação
UserParameter=db2.tx_log_usage, sudo su - db2ecp -c "db2 -x 'SELECT DECIMAL(TOTAL_LOG_USED_KB * 100.0 / TOTAL_LOG_AVAILABLE_KB, 10,2) FROM SYSIBMADM.LOG_UTILIZATION;'"

### --- Licenciamento ---
UserParameter=db2.product_name, sudo su - db2ecp -c "db2licm -l | grep \"Product name\" | awk -F': ' '{print $2}'"
UserParameter=db2.license_expiry, sudo su - db2ecp -c "db2licm -l | grep \"Expiry date\" | awk -F': ' '{print $2}'"
### 1. Pré-requisitos
O usuário do Zabbix precisa de permissão para executar comandos como o usuário da instância DB2 (ex: `db2ecp`). Adicione ao seu arquivo `sudoers`:

```bash
zabbix ALL=(ALL) NOPASSWD:ALL

# Exemplo: Coleta de tamanho do banco
UserParameter=db2.db_size, sudo su - db2ecp -c "db2 -x 'SELECT SUM(TBSP_USED_PAGES * 4096) / 1024 AS TOTAL_SIZE_GB FROM SYSIBMADM.TBSP_UTILIZATION;'"

# Exemplo: Status do Backup (1 = Sucesso)
UserParameter=db2.backup_status, sudo su - db2ecp -c "db2 -x 'SELECT CASE WHEN COUNT(*) > 0 THEN 1 ELSE 0 END FROM SYSIBMADM.DB_HISTORY WHERE SQLCODE = 0;'"
