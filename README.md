# 📊 Zabbix Monitoring Template for IBM DB2

![Zabbix](https://img.shields.io/badge/Zabbix-6.0%2B-red)
![DB2](https://img.shields.io/badge/IBM%20DB2-Monitoring-blue)
![License](https://img.shields.io/badge/License-MIT-green)

Este repositório contém um template personalizado para monitoramento de instâncias **IBM DB2** via Zabbix. 

🚀 **Diferencial:** Este projeto nasceu da necessidade real de monitorar o DB2 em ambientes de produção, dado que não existem templates nativos robustos na comunidade Zabbix.

## 🔍 Funcionalidades Monitoradas

O template utiliza `UserParameters` para coletar métricas essenciais diretamente do banco de dados:

* **Disponibilidade:** Status da instância (Up/Down) e tempo de resposta de query.
* **Performance:** Uso de CPU e Memória RAM exclusivo dos processos DB2.
* **Armazenamento:** Tamanho total do banco (GB), capacidade e uso de logs de transação.
* **Conexões:** Contagem de usuários ativos e status de conexões.
* **Saúde & Backup:** Status do último backup e contagem de Slow Queries (> 5s).
* **Licenciamento:** Nome do produto, expiração e tipo de licença.

## 🛠️ Configuração e Instalação

### 1. Pré-requisitos
O usuário do Zabbix precisa de permissão para executar comandos como o usuário da instância DB2 (ex: `db2ecp`). Adicione ao seu arquivo `sudoers`:

```bash
zabbix ALL=(ALL) NOPASSWD:ALL
