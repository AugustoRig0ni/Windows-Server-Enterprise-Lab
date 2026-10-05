# Windows Server Enterprise Lab

Laboratório de Windows Server que estou montando no Hyper-V para simular a infraestrutura de uma empresa pequena, a **LABTECH**.

A ideia é praticar administração de servidores, redes e serviços de infraestrutura, começando por Active Directory, DNS e DHCP e evoluindo posteriormente para permissões, GPO, PowerShell e troubleshooting.

Vou construindo aos poucos e só avanço depois de validar a etapa anterior.

**Onde estou:** AD, DNS e DHCP estão implementados e validados no servidor. O DHCP foi testado com o `CLIENT-01`, uma VM no Hyper-V, que recebeu endereço via DHCP e ingressou no domínio `ad.labtech.lab`. OUs, grupos, usuários, File Server, GPOs e scripts ainda não foram implementados.

**Sobre a documentação:** usei IA para estruturar a documentação deste projeto. A implementação e as validações no laboratório foram feitas por mim.

---

## Documentação

| Arquivo                | O que tem                               |
| ---------------------- | --------------------------------------- |
| `docs/planejamento.md` | O plano inicial do laboratório          |
| `docs/arquitetura.md`  | Como o ambiente está montado atualmente |
| `docs/decisoes.md`     | Por que escolhi cada configuração       |

Os outros documentos, como troubleshooting e detalhes de serviços, serão adicionados conforme forem sendo implementados e validados.

---

## Como está o ambiente

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
       192.168.10.10       192.168.10.101
             │
      ┌──────┼──────┐
     AD DS  DNS   DHCP
             │
      ad.labtech.lab
```

| Item          | Valor                                                                    |
| ------------- | ------------------------------------------------------------------------ |
| Servidor      | `SRV-DC01`, Windows Server 2022 Standard Evaluation (Desktop Experience) |
| Domínio       | `ad.labtech.lab`                                                         |
| Rede          | `192.168.10.0/24`, isolada, sem gateway                                  |
| DNS           | `192.168.10.10`                                                          |
| IPs estáticos | `.1` a `.99` (o servidor é o `.10`)                                      |
| Pool DHCP     | `.100` a `.200` (101 endereços)                                          |
| Faixa livre   | `.201` a `.254` (expansão futura)                                        |

### Estrutura de OUs e grupos planejada

```text
LABTECH
├── Usuarios
│   ├── TI
│   ├── RH
│   └── Financeiro
├── Computadores
└── Grupos
    ├── GRP-TI
    ├── GRP-RH
    └── GRP-FINANCEIRO
```

---

## Andamento

| Etapa                                    | Status                                  |
| ---------------------------------------- | --------------------------------------- |
| Hyper-V e switch `LAB-SWITCH`            | Feito                                   |
| Windows Server 2022 com IP estático      | Feito                                   |
| Active Directory (`ad.labtech.lab`)      | Feito                                   |
| DNS                                      | Feito e validado com `dcdiag /test:dns` |
| DHCP (escopo `LABTECH-LAN`)              | Feito e validado com o cliente          |
| `CLIENT-01` recebendo IP via DHCP        | Feito                                   |
| `CLIENT-01` no domínio                   | Feito e validado                        |
| Canal seguro entre cliente e domínio     | Feito e validado                        |
| OUs, grupos e usuários                   | Não feito                               |
| File Server e permissões NTFS            | Não feito                               |
| Group Policy                             | Não feito                               |
| Script PowerShell de criação de usuários | Não feito                               |
| Cenários de troubleshooting              | Não feito                               |

### Comandos usados na validação até agora

```powershell
# No SRV-DC01
nslookup ad.labtech.lab
dcdiag /test:dns
Get-DhcpServerv4Scope
Get-DhcpServerv4OptionValue -ScopeId 192.168.10.0
Get-DhcpServerv4Lease -ScopeId 192.168.10.0

# No CLIENT-01
ipconfig /all
Test-NetConnection 192.168.10.10 -Port 53
Resolve-DnsName ad.labtech.lab -Server 192.168.10.10 -Type A
Get-ComputerInfo | Select-Object CsName, CsDomain, CsDomainRole
Test-ComputerSecureChannel -Verbose
whoami
```

### Validações realizadas

- `SRV-DC01` recebeu IP estático `192.168.10.10`.
- O domínio `ad.labtech.lab` foi criado e validado.
- O DNS foi validado com `dcdiag /test:dns`.
- O escopo DHCP `LABTECH-LAN` foi criado e ativado.
- O `CLIENT-01` recebeu `192.168.10.101` automaticamente via DHCP.
- O cliente recebeu `192.168.10.10` como servidor DNS.
- A resolução direta de `ad.labtech.lab` retornou `192.168.10.10`.
- A porta TCP 53 do servidor DNS foi validada a partir do cliente.
- O `CLIENT-01` ingressou no domínio `ad.labtech.lab`.
- O canal seguro entre o cliente e o domínio foi validado com sucesso.
- O fuso horário e o horário do servidor e do cliente foram alinhados antes do ingresso no domínio.

---

## Problemas que apareceram

Aqui vou registrar somente problemas que realmente acontecerem durante a implementação: o sintoma, como investiguei, a causa e a correção.

| Data | Problema                | Causa | Solução |
| ---- | ----------------------- | ----- | ------- |
| —    | Nenhum registrado ainda | —     | —       |

Os problemas ocorridos durante a configuração também podem ser documentados posteriormente como lições aprendidas, quando houver informações úteis para reproduzir ou entender a situação.

---

## Cenários que vou simular

Além dos problemas reais, pretendo quebrar o ambiente de propósito para treinar diagnóstico. Quando fizer isso, vou deixar claro na documentação que foi uma simulação.

* Cliente com DNS errado
* Cliente sem IP do DHCP
* GPO que não aplica
* Permissão NTFS errada e acesso negado a compartilhamento
* Cliente fora do domínio
* Falha de resolução de nomes
* Cliente sem conseguir falar com o Domain Controller

Cada caso seguirá o mesmo roteiro:

```text
Sintoma
   ↓
Hipótese
   ↓
Diagnóstico
   ↓
Comandos utilizados
   ↓
Causa
   ↓
Correção
   ↓
Validação
```

---

## O que falta fazer

* Criar a estrutura de OUs e validar no Active Directory
* Criar os grupos de segurança
* Criar os usuários
* Mover o `CLIENT-01` para a OU `Computadores`
* Criar o File Server e validar as permissões
* Aplicar GPOs
* Escrever o script `create-users.ps1` para criação de usuários a partir de CSV, sem senhas armazenadas no arquivo
* Executar os cenários de troubleshooting
* Manter a documentação e as evidências em dia

---

## Estrutura do repositório

```text
Windows-Server-Enterprise-Lab/
├── README.md
├── .gitignore
└── docs/
    ├── planejamento.md
    ├── arquitetura.md
    └── decisoes.md
```

Mais pastas e arquivos, como `scripts/`, `screenshots/` e outros documentos, serão adicionados quando houver conteúdo.

---

## Tecnologias

**Já em uso:**

* Windows Server 2022
* Active Directory
* DNS
* DHCP
* Hyper-V
* Git
* GitHub

**Previstas:**

* Group Policy
* NTFS
* SMB
* PowerShell

O laboratório é um ambiente de estudo e as configurações deverão ser adaptadas antes de qualquer utilização em um ambiente de produção.