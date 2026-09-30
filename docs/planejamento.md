# Planejamento — Windows Server Enterprise Lab

> Plano inicial, escrito em 30/09/2026. Não atualizo este documento conforme o projeto avança. O estado atual está no README e em arquitetura.md.

## Por que fiz este projeto

- Colocar em prática o que aprendi em cursos e videoaulas.
- Trabalhar num ambiente de administração que vai evoluindo aos poucos.
- Ser hands-on e ampliar meu repertório prático na criação e administração de servidores Windows.

## Como vou trabalhar

Uma etapa por vez, documentando e testando, e só passo para a próxima quando tudo estiver validado.

---

## 2. Cenário proposto

O laboratório simulará uma empresa chamada **LABTECH**, com diferentes setores e usuários.

A infraestrutura será centralizada em um servidor Windows Server responsável inicialmente pelos serviços de:

* Active Directory Domain Services (AD DS)
* DNS
* DHCP
* File Server
* Gerenciamento de usuários e grupos
* Group Policy (GPO)
* Automação utilizando PowerShell

Posteriormente, serão realizados testes a partir de uma máquina cliente ingressada no domínio.

---

## 3. Ambiente planejado

### Servidor

| Item     | Configuração                            |
| -------- | --------------------------------------- |
| Hostname | `SRV-DC01`                              |
| Sistema  | Windows Server 2022 Standard Evaluation |
| Função   | Domain Controller                       |
| Domínio  | `ad.labtech.lab`                        |
| IP       | `192.168.10.10`                         |
| Máscara  | `255.255.255.0`                         |
| Gateway  | Não utilizado inicialmente              |
| DNS      | `192.168.10.10`                         |

### Rede

Será utilizada uma rede **Internal Switch** do Hyper-V chamada:

`LAB-SWITCH`

A rede utilizará:

`192.168.10.0/24`

O endereçamento será organizado da seguinte maneira:

| Faixa                | Finalidade                           |
| -------------------- | ------------------------------------ |
| `192.168.10.1–99`    | Infraestrutura e endereços estáticos |
| `192.168.10.100–200` | Clientes via DHCP                    |
| `192.168.10.201–254` | Reserva para expansão                |

---

## 4. Estrutura do Active Directory

Será criada a seguinte estrutura organizacional:

```text
LABTECH
│
├── Usuarios
│   ├── TI
│   ├── RH
│   └── Financeiro
│
├── Computadores
│
└── Grupos
```

Serão criados grupos de segurança para controle de acesso:

* `GRP-TI`
* `GRP-RH`
* `GRP-FINANCEIRO`

Também serão criados usuários de teste para representar os diferentes setores.

---

## 5. Serviços planejados

### Active Directory

Responsável pelo gerenciamento centralizado de identidades, computadores, grupos e autenticação.

### DNS

Responsável pela resolução de nomes da rede e pelo suporte à localização dos serviços do Active Directory.

### DHCP

Responsável pela distribuição automática de configurações de rede aos computadores clientes.

Escopo planejado:

`192.168.10.100–192.168.10.200`

Configurações distribuídas:

* Máscara: `255.255.255.0`
* DNS: `192.168.10.10`
* Domínio DNS: `ad.labtech.lab`
* Gateway: não utilizado inicialmente

### File Server

Serão criados compartilhamentos separados por setor:

```text
D:\Shares\
├── TI
├── RH
├── Financeiro
└── Publico
```

Serão aplicadas permissões de compartilhamento e NTFS de acordo com os grupos de segurança.

### Group Policy

Serão implementadas políticas para simular um ambiente corporativo, incluindo:

* Política de senha
* Configurações de segurança
* Configurações de usuários e computadores
* Mapeamento de unidades de rede
* Outras políticas administrativas

---

## 6. Máquina cliente

Será criada uma máquina virtual `CLIENT-01`.

O cliente deverá:

1. Receber endereço IP através do DHCP.
2. Utilizar `SRV-DC01` como servidor DNS.
3. Resolver o domínio `ad.labtech.lab`.
4. Ingressar no domínio.
5. Autenticar utilizando uma conta do Active Directory.
6. Receber as políticas de grupo.
7. Acessar os compartilhamentos de acordo com suas permissões.

---

## 7. Automação

Será desenvolvido um conjunto de scripts PowerShell para automatizar tarefas administrativas.

O primeiro script deverá permitir a criação de usuários a partir de um arquivo CSV.

Fluxo planejado:

```text
usuarios.csv
     ↓
PowerShell
     ↓
Criação dos usuários
     ↓
Associação aos grupos
     ↓
Registro das operações em log
```

Os scripts deverão realizar validações para evitar a criação duplicada de contas e facilitar o diagnóstico de erros.

---

## 8. Troubleshooting

O laboratório também será utilizado para simular problemas comuns encontrados em ambientes Windows.

Cenários planejados:

* Cliente configurado com DNS incorreto
* Cliente sem endereço DHCP
* Falha na aplicação de GPO
* Problemas de permissões NTFS
* Problemas de acesso a compartilhamentos
* Cliente fora do domínio
* Problemas de resolução de nomes
* Falhas de comunicação com o Domain Controller

Cada cenário deverá ser documentado com:

1. Sintoma
2. Hipótese
3. Diagnóstico
4. Comandos/ferramentas utilizados
5. Causa
6. Correção
7. Validação

---

## 9. Documentação e versionamento

O projeto será documentado progressivamente no GitHub.

A documentação será organizada por componente:

```text
docs/
├── planejamento.md
├── arquitetura.md
├── active-directory.md
├── dhcp.md
├── file-server.md
├── gpo.md
└── troubleshooting.md
```

Screenshots serão utilizados como evidência das principais configurações e validações.

Máquinas virtuais, discos virtuais, ISOs e outros arquivos grandes do laboratório não serão armazenados no repositório.

---

## 10. Critérios de conclusão

O projeto será considerado funcional quando:

* [ ] Active Directory estiver estruturado
* [ ] DNS estiver funcionando
* [ ] DHCP estiver distribuindo configurações corretamente
* [ ] CLIENT-01 estiver ingressado no domínio
* [ ] Usuários e grupos estiverem configurados
* [ ] Compartilhamentos estiverem funcionando
* [ ] Permissões NTFS estiverem validadas
* [ ] GPOs estiverem aplicadas
* [ ] Automação PowerShell estiver funcionando
* [ ] Cenários de troubleshooting tiverem sido executados
* [ ] Documentação estiver atualizada
* [ ] Resultados estiverem registrados no GitHub

---

## 11. Tecnologias utilizadas

* Windows Server 2022
* Active Directory Domain Services
* DNS
* DHCP
* Group Policy
* NTFS
* SMB
* PowerShell
* Hyper-V
* Git
* GitHub