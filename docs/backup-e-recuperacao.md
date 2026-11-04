# Backup e recuperação

O backup do `SRV-DC01` usa o recurso **Windows Server Backup**, com um disco virtual dedicado.

## Disco de backup

| Item | Valor |
| ---- | ----- |
| Tipo | VHDX dinâmico, de aproximadamente 40 GB |
| Uso | Exclusivamente backups |
| Unidade | `E:` |
| Rótulo | `BACKUP` |
| Sistema de arquivos | NTFS |

O disco foi conectado à VM `SRV-DC01` e formatado no Windows.

## Instalação do recurso

O recurso Windows Server Backup foi instalado e a instalação foi confirmada com:

```powershell
Get-WindowsFeature Windows-Server-Backup
```

O console do Windows Server Backup foi aberto normalmente.

## Configuração do backup

| Item | Valor |
| ---- | ----- |
| Destino | `E:\` (`BACKUP`) |
| Itens selecionados | System State e Local Disk (`C:`) |
| Opção VSS | VSS Copy Backup |

Nenhum arquivo foi excluído manualmente.

## Resultado do primeiro backup

| Item | Valor |
| ---- | ----- |
| Status no console | `SUCCESSFUL` |
| Dados transferidos | 9,08 GB |
| Local | Disco `E:` |

O conjunto de backup foi conferido com:

```powershell
Get-WBBackupSet
```

Resultado registrado:

| Campo | Valor |
| ----- | ----- |
| `VersionId` | `11/03/2026-12:32` (formato mês/dia/ano) |
| `BackupTime` | `03/11/2026 09:32:57` |
| `BackupTarget` | `E:` |
| `RecoverableItems` | Volumes, SystemState, Applications, Files |
| Volume | Local disk (`C:`) |
| Application | FRS, AD, Registry |
| `VssBackupOption` | `VssCopyBackup` |

## Teste de recuperação

Foi executado o assistente **Recover** do Windows Server Backup.

Procedimento:

1. Servidor local selecionado.
2. Backup de 03/11/2026 selecionado.
3. Local Disk (`C:`) selecionado.
4. Recuperação direcionada para **outra localização**: a pasta `C:\Teste-Restauracao`, criada para esse fim.

Resultado:

- Status: `Completed`.
- A estrutura de pastas do volume `C:` foi recuperada na pasta alternativa.

O teste confirmou que o backup pode ser lido e recuperado sem sobrescrever o volume original.

## Limitações

- **Mesmo host físico:** o VHDX de backup está no mesmo computador que hospeda a VM. A configuração serve para o laboratório, mas não equivale a uma estratégia de backup externa contra a perda do host.
- **Escopo do teste:** o teste de recuperação cobriu a recuperação de arquivos do volume `C:` para outra pasta. O System State e o Active Directory não tiveram restauração completa testada.
- **Rotina:** o backup foi pontual, executado e verificado uma vez. Não há rotina recorrente de backup documentada.
