# Evoluções futuras

A versão 1 está documentada nos demais arquivos. O plano é **reconstruir o laboratório do zero** e evoluí-lo progressivamente. Nada abaixo foi implementado ainda.

## pfSense

- Implantar um firewall dedicado em VM.
- Separar LAN e WAN e criar segmentação de rede.
- Criar regras de firewall e controlar o tráfego entre os segmentos.
- Testar bloqueios e liberações.

## Zabbix

- Implantar monitoramento centralizado.
- Monitorar `SRV-DC01` e `CLIENT-01`: CPU, RAM, disco e disponibilidade.
- Monitorar serviços essenciais.
- Criar dashboards e alertas.

## PowerShell

Criar scripts para:

- Criação automatizada de usuários no Active Directory.
- Inserção dos usuários nas OUs corretas.
- Associação automática aos grupos.
- Criação de pastas departamentais.
- Configuração automatizada de permissões.
- Relatórios administrativos.
- Tarefas recorrentes de manutenção.

## Pendências da versão 1

- Teste de restauração completa do System State e do Active Directory.
- Rotina recorrente de backup (hoje há apenas um backup pontual).
- Cópia do backup em local externo ao host físico.
- Cenários de troubleshooting simulados, documentados com sintoma, hipótese, diagnóstico, causa, correção e validação.
