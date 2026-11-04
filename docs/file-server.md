# File Server e compartilhamentos

O servidor de arquivos roda no `SRV-DC01`. O acesso é separado por departamento, com grupos de segurança do AD.

## Estrutura de pastas

```text
C:\Compartilhamentos\
├── TI\
├── RH\
├── Financeiro\
└── Publico\
```

## Compartilhamentos SMB

| Compartilhamento | Pasta |
| ---------------- | ----- |
| `\\SRV-DC01\TI` | `C:\Compartilhamentos\TI` |
| `\\SRV-DC01\RH` | `C:\Compartilhamentos\RH` |
| `\\SRV-DC01\Financeiro` | `C:\Compartilhamentos\Financeiro` |
| `\\SRV-DC01\Publico` | `C:\Compartilhamentos\Publico` |

## Modelo de permissões

A camada **SMB** libera o acesso ao compartilhamento de forma ampla, e a camada **NTFS** controla de fato quem entra em cada pasta. Quando as duas se combinam, vale a mais restritiva.

### Permissões SMB (todos os compartilhamentos)

| Identidade | Permissão |
| ---------- | --------- |
| `Authenticated Users` | Change |
| `Administrators` | Full Control |

### Permissões NTFS

Pastas departamentais (`TI`, `RH`, `Financeiro`):

| Identidade | Permissão |
| ---------- | --------- |
| `SYSTEM` | Full Control |
| `BUILTIN\Administrators` | Full Control |
| Grupo do departamento (`GRP TI`, `GRP RH` ou `GRP Financeiro`) | Modify |

Pasta `Publico`:

| Identidade | Permissão |
| ---------- | --------- |
| `SYSTEM` | Full Control |
| `BUILTIN\Administrators` | Full Control |
| `LABTECH\Domain Users` | Modify |

As permissões são herdadas por arquivos e subpastas.

## Resultado esperado e validado

| Usuário | Publico | TI | RH | Financeiro |
| ------- | :-----: | :-: | :-: | :--------: |
| João Silva (TI) | Acesso | Acesso | Bloqueado | Bloqueado |
| Maria Azevedo (RH) | Acesso | Bloqueado | Acesso | Bloqueado |
| Carlos Pedroso (Financeiro) | Acesso | Bloqueado | Bloqueado | Acesso |

Nas pastas permitidas, cada usuário criou e excluiu arquivos. Os testes foram repetidos depois da reinicialização do servidor e do cliente, com o mesmo resultado. Resultados completos em [testes-e-validacoes.md](testes-e-validacoes.md).

## Integração com GPO

As unidades de rede `P:`, `T:`, `R:` e `F:` apontam para estes compartilhamentos e são mapeadas por GPO, conforme o grupo do usuário (ver [gpo.md](gpo.md)).

## Observação

O servidor de arquivos está no mesmo servidor que o controlador de domínio. Em produção, as funções seriam separadas (ver [decisoes.md](decisoes.md)).
