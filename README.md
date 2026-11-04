# Windows Server Enterprise Lab

Laboratório de infraestrutura Windows Server que montei no Hyper-V para simular a rede de uma empresa pequena, a **LABTECH**. O ambiente reúne Active Directory, DNS, DHCP, servidor de arquivos com permissões por departamento, GPOs e backup com teste de recuperação.

**Status:** versão 1 concluída e validada (novembro de 2026). O próximo passo é reconstruir o laboratório do zero e evoluí-lo com firewall dedicado (pfSense), monitoramento (Zabbix) e automação em PowerShell. Essas evoluções ainda **não foram implementadas** (veja [`docs/evolucoes-futuras.md`](docs/evolucoes-futuras.md)).

**Sobre a documentação:** usei IA para estruturar a documentação deste projeto. A implementação e as validações no laboratório foram feitas por mim.

---

## O que foi implementado

| Área | O que foi feito | Documento |
| ---- | --------------- | --------- |
| Virtualização e rede | Hyper-V, switch interno `LAB-SWITCH`, rede isolada `192.168.10.0/24` | [`arquitetura.md`](docs/arquitetura.md) |
| Active Directory | Domínio `ad.labtech.lab`, OUs, 3 usuários, 3 grupos de segurança, cliente no domínio | [`active-directory.md`](docs/active-directory.md) |
| DNS e DHCP | DNS integrado ao AD, escopo `LABTECH-LAN` autorizado, cliente recebendo endereço | [`dns-dhcp.md`](docs/dns-dhcp.md) |
| File Server | 4 compartilhamentos SMB, permissões NTFS por grupo | [`file-server.md`](docs/file-server.md) |
| Group Policy | Mapeamento de unidades por grupo e política de segurança para estações | [`gpo.md`](docs/gpo.md) |
| Backup | Windows Server Backup (System State + volume C:) e teste de recuperação | [`backup-e-recuperacao.md`](docs/backup-e-recuperacao.md) |
| Validações | Testes de AD, DNS, DHCP, permissões, GPO, reinicialização e backup | [`testes-e-validacoes.md`](docs/testes-e-validacoes.md) |

---

## Ambiente

```text
                 HOST WINDOWS
                      │
               Hyper-V / Internal
                      │
                 LAB-SWITCH
                192.168.10.0/24
                      │
             ┌────────┴────────┐
             │                 │
         SRV-DC01          CLIENT-01
       192.168.10.10       192.168.10.101 (DHCP)
             │
   ┌────┬────┼─────┐
 AD DS  DNS  DHCP  File Server
             │
      ad.labtech.lab
```

| Item | Valor |
| ---- | ----- |
| Servidor | `SRV-DC01`, Windows Server 2022 Standard Evaluation (Desktop Experience), 4 GB de RAM |
| Funções do servidor | AD DS, DNS, DHCP e File Server |
| Cliente | Hostname `DESKTOP-81PP8GF`, identificado como `CLIENT-01` neste projeto; IP por DHCP |
| Domínio | `ad.labtech.lab` (NetBIOS `LABTECH`) |
| Rede | `192.168.10.0/24`, isolada, sem gateway |
| Pool DHCP | `192.168.10.100` a `192.168.10.200` |

---

## Resumo das validações

- Três usuários testados no cliente: cada um acessa a pasta `Publico` e a do próprio departamento, cria e exclui arquivos, e é bloqueado nas pastas dos outros departamentos.
- Unidades de rede mapeadas corretamente por grupo (`P:` para todos, mais `T:`, `R:` ou `F:`).
- Política de segurança das estações aplicada e conferida com `gpresult`.
- Servidor e cliente reiniciados: serviços saudáveis e acessos revalidados.
- Backup concluído com sucesso (status `SUCCESSFUL`) e recuperação do volume `C:` para outra pasta (status `Completed`).

Detalhes em [`docs/testes-e-validacoes.md`](docs/testes-e-validacoes.md).

---

## Problemas encontrados

Um problema real foi registrado: durante a preparação do cliente para o domínio, ele apresentava horário americano, enquanto o computador físico estava no horário de Brasília. O horário do cliente foi ajustado e o problema foi resolvido. A causa técnica original não foi registrada. Detalhes em [`docs/testes-e-validacoes.md`](docs/testes-e-validacoes.md).

---

## Limitações conhecidas

- O disco de backup (VHDX) está no mesmo computador físico que hospeda a VM. Serve para o laboratório, mas não protege contra perda do host.
- O backup foi pontual, sem rotina recorrente. O teste de recuperação cobriu arquivos do volume `C:` em outra pasta. O System State e o Active Directory não tiveram restauração completa testada.
- Um único servidor concentra AD DS, DNS, DHCP e File Server. Em produção, as funções seriam separadas.
- A rede é isolada, sem acesso à Internet.
- O Windows Server Evaluation tem prazo de 180 dias (instalado em 30/09/2026).
- **Não implementados:** firewall dedicado, segmentação de rede, monitoramento centralizado e automação de usuários.

---

## Documentação

| Arquivo | Conteúdo |
| ------- | -------- |
| [`docs/planejamento.md`](docs/planejamento.md) | Plano inicial (documento histórico, não atualizado) |
| [`docs/arquitetura.md`](docs/arquitetura.md) | Visão geral do ambiente |
| [`docs/decisoes.md`](docs/decisoes.md) | Decisões de projeto e motivos |
| [`docs/active-directory.md`](docs/active-directory.md) | Domínio, OUs, usuários e grupos |
| [`docs/dns-dhcp.md`](docs/dns-dhcp.md) | Configuração de DNS e DHCP |
| [`docs/file-server.md`](docs/file-server.md) | Compartilhamentos e permissões |
| [`docs/gpo.md`](docs/gpo.md) | Políticas de grupo |
| [`docs/backup-e-recuperacao.md`](docs/backup-e-recuperacao.md) | Backup e teste de recuperação |
| [`docs/testes-e-validacoes.md`](docs/testes-e-validacoes.md) | Resultados dos testes |
| [`docs/evolucoes-futuras.md`](docs/evolucoes-futuras.md) | O que ainda não foi feito |

---

## Estrutura do repositório

```text
Windows-Server-Enterprise-Lab/
├── README.md
├── .gitignore
└── docs/
    ├── planejamento.md
    ├── arquitetura.md
    ├── decisoes.md
    ├── active-directory.md
    ├── dns-dhcp.md
    ├── file-server.md
    ├── gpo.md
    ├── backup-e-recuperacao.md
    ├── testes-e-validacoes.md
    └── evolucoes-futuras.md
```

---

## Tecnologias

Windows Server 2022, Hyper-V, Active Directory Domain Services, DNS, DHCP, SMB, NTFS, Group Policy, Windows Server Backup, Git e GitHub.

O laboratório é um ambiente de estudo e as configurações devem ser adaptadas antes de qualquer uso em produção.
