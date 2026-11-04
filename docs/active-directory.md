# Active Directory

## Domínio

| Item | Valor |
| ---- | ----- |
| Domínio | `ad.labtech.lab` |
| NetBIOS | `LABTECH` |
| Controlador de domínio | `SRV-DC01` (`192.168.10.10`) |
| DNS | Integrado, no próprio `SRV-DC01` |

O servidor usa o próprio endereço como DNS, requisito para o funcionamento do AD DS (ver [dns-dhcp.md](dns-dhcp.md)).

## Estrutura de OUs

Raiz: `OU=LABTECH,DC=ad,DC=labtech,DC=lab`

```text
Domain Controllers          (OU padrão do Active Directory)
LABTECH
├── Usuários
│   ├── TI
│   ├── RH
│   └── Financeiro
├── Computadores
└── Grupos
```

A estrutura foi confirmada no ambiente com `Get-ADOrganizationalUnit -Filter *`. O nome da OU de usuários é `Usuários`, com acento.

| OU | Conteúdo |
| -- | -------- |
| `Usuários\TI`, `Usuários\RH`, `Usuários\Financeiro` | Contas dos usuários de cada departamento |
| `Computadores` | Estações do domínio (aqui está o computador do `CLIENT-01`) |
| `Grupos` | Grupos de segurança |

A organização por OUs permite aplicar GPOs por escopo (ver [gpo.md](gpo.md)).

## Usuários

| Nome | Login | Departamento |
| ---- | ----- | ------------ |
| João Silva | `joao.silva` | TI |
| Maria Azevedo | `maria.azevedo` | RH |
| Carlos Pedroso | `carlos.pedroso` | Financeiro |

## Grupos de segurança

Grupos globais de segurança, um por departamento:

| Grupo | Membro |
| ----- | ------ |
| `GRP TI` | `joao.silva` |
| `GRP RH` | `maria.azevedo` |
| `GRP Financeiro` | `carlos.pedroso` |

Esses grupos controlam o acesso às pastas (NTFS, ver [file-server.md](file-server.md)) e o mapeamento de unidades de rede (GPO).

## Computador cliente

| Item | Valor |
| ---- | ----- |
| Hostname | `DESKTOP-81PP8GF` |
| Identificação lógica no projeto | `CLIENT-01` |
| Endereço | `192.168.10.101` (DHCP) |
| Localização no AD | OU `Computadores` do laboratório |

O computador ingressou no domínio `ad.labtech.lab` e depois teve seu objeto movido para a OU `Computadores`, que recebe a GPO de segurança das estações.

## Validações

- Usuários criados e grupos criados, com cada usuário associado ao grupo do seu departamento.
- Estrutura de OUs organizada.
- Cliente integrado ao domínio.
- Canal seguro do cliente com o domínio validado com `Test-ComputerSecureChannel`.
- Após reinicialização do servidor e do cliente, o AD DS apareceu saudável no Server Manager.

Os resultados completos estão em [testes-e-validacoes.md](testes-e-validacoes.md).

## Evidências

Estrutura de OUs:

![Estrutura de OUs no Active Directory](../screenshots/ESTRUTURA-UOS.png)

Usuários e grupos:

![Usuários e grupos no Active Directory](../screenshots/USUARIOS-GRUPOS.png)
