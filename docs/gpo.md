# Políticas de grupo (GPO)

Foram criadas e validadas duas GPOs.

| GPO | Vinculada a | Tipo |
| --- | ----------- | ---- |
| `GPO-Mapeamento-Unidades` | OU `LABTECH` | Configuração de usuário |
| `GPO-Seguranca-Estacoes` | OU `LABTECH → Computadores` | Configuração de computador |

## GPO-Mapeamento-Unidades

Configurada em `User Configuration → Preferences → Windows Settings → Drive Maps`.

| Unidade | Destino | Aplicada a |
| ------- | ------- | ---------- |
| `P:` | `\\SRV-DC01\Publico` | Todos os usuários |
| `T:` | `\\SRV-DC01\TI` | Membros de `GRP TI` |
| `R:` | `\\SRV-DC01\RH` | Membros de `GRP RH` |
| `F:` | `\\SRV-DC01\Financeiro` | Membros de `GRP Financeiro` |

Os mapeamentos departamentais usam **Item-Level Targeting**, associado ao grupo de segurança correspondente. Uma única GPO entrega a cada usuário só a unidade do seu departamento.

Resultado esperado e validado:

| Usuário | Unidades |
| ------- | -------- |
| João Silva | `P:` e `T:` |
| Maria Azevedo | `P:` e `R:` |
| Carlos Pedroso | `P:` e `F:` |

A validação foi feita após atualização de políticas e novo logon.

## GPO-Seguranca-Estacoes

Vinculada à OU `Computadores`, onde está o computador do `CLIENT-01`.

| Configuração | Valor |
| ------------ | ----- |
| `Interactive logon: Machine inactivity limit` | 900 segundos (15 minutos) |
| `Windows Defender Firewall with Advanced Security` | Firewall ativado nos perfis Domain, Private e Public |

### Validação

A aplicação foi validada no cliente com:

```powershell
gpupdate /force
gpresult /r /scope computer
```

O cliente foi reiniciado entre os dois comandos. O resultado do `gpresult` foi usado para confirmar a aplicação da política de computador.

## Observação

As permissões de acesso às pastas não vêm das GPOs: são NTFS e SMB (ver [file-server.md](file-server.md)). A GPO só mapeia as unidades.
