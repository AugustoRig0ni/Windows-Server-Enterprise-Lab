# Testes e validações

Resumo dos testes executados na versão 1 do laboratório. Cada área tem detalhes no documento correspondente.

## Active Directory

| Teste | Resultado |
| ----- | --------- |
| Usuários criados | Concluído |
| Grupos criados e membros associados | Concluído |
| Estrutura de OUs organizada (`Get-ADOrganizationalUnit -Filter *`) | Confirmada, ver [active-directory.md](active-directory.md) |
| Cliente integrado ao domínio | Concluído |
| Canal seguro do cliente com o domínio (`Test-ComputerSecureChannel`) | Validado com sucesso |

## DNS e DHCP

| Teste | Resultado |
| ----- | --------- |
| Resolução de nomes (`nslookup`, `Resolve-DnsName`) | Validada |
| Diagnóstico DNS (`dcdiag /test:dns`) | Executado com sucesso |
| Porta TCP 53 do servidor, a partir do cliente (`Test-NetConnection`) | Respondeu |
| Escopo DHCP autorizado e ativo (`Get-DhcpServerv4Scope`) | Validado |
| Opções DHCP (`Get-DhcpServerv4OptionValue`) | Validadas |
| Cliente recebeu `192.168.10.101` por DHCP (`Get-DhcpServerv4Lease`, `ipconfig /all`) | Validado |
| Horário do cliente ajustado antes do ingresso no domínio | Concluído (ver Problemas encontrados) |

## File Server

Testes com os três usuários.

| Usuário | Publico | Pasta do departamento | Outras pastas |
| ------- | :-----: | :-------------------: | :-----------: |
| João Silva (TI) | Acesso | Acesso (TI) | RH e Financeiro bloqueadas |
| Maria Azevedo (RH) | Acesso | Acesso (RH) | TI e Financeiro bloqueadas |
| Carlos Pedroso (Financeiro) | Acesso | Acesso (Financeiro) | TI e RH bloqueadas |

Para os três usuários: criação e exclusão de arquivos validadas. **Resultado:** todos os testes de permissão foram validados.

## Group Policy

| Teste | Resultado |
| ----- | --------- |
| Mapeamento de unidades após atualização de políticas e novo logon | João: `P:` e `T:`; Maria: `P:` e `R:`; Carlos: `P:` e `F:` |
| GPO de segurança das estações (`gpupdate /force`, reinicialização, `gpresult /r /scope computer`) | Aplicação validada |

## Reinicialização

O servidor e o cliente foram reiniciados. Depois disso:

| Verificação | Resultado |
| ----------- | --------- |
| AD DS, DHCP, DNS e File and Storage Services no Server Manager | Saudáveis |
| Acesso às pastas | Validado novamente |
| Criação e exclusão de arquivos | Validadas |
| Restrições de acesso por departamento | Validadas |

## Backup e recuperação

| Teste | Resultado |
| ----- | --------- |
| Instalação do Windows Server Backup (`Get-WindowsFeature Windows-Server-Backup`) | Confirmada |
| Primeiro backup (System State + `C:`) | `SUCCESSFUL`, 9,08 GB |
| Conjunto de backup (`Get-WBBackupSet`) | Registrado em `E:` |
| Recuperação do volume `C:` para `C:\Teste-Restauracao` | `Completed` |

## Problemas encontrados

Um problema real foi encontrado e resolvido. Estão registrados apenas os fatos confirmados.

| Item | Registro |
| ---- | -------- |
| Quando | Durante a preparação do cliente para o ambiente de domínio (data não registrada) |
| Sintoma | O cliente apresentava horário americano, enquanto o computador físico estava configurado no horário de Brasília |
| Causa | Não registrada: a causa técnica original não foi identificada |
| Correção | O horário do cliente foi ajustado |
| Validação | O problema foi resolvido após o ajuste |

**Lição:** conferir o fuso horário e o horário do cliente antes de ingressá-lo no domínio.

## Não testado nesta versão

- Restauração completa do System State e do Active Directory.
- Rotina recorrente de backup (o backup foi pontual).
- Backup em local externo ao host físico.
- Cenários de falha provocados de propósito (por exemplo, DNS incorreto no cliente ou GPO que não aplica).
