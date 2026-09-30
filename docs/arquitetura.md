# Arquitetura da Infraestrutura

## 1. Visão geral

Este laboratório simula uma infraestrutura de TI corporativa de pequeno porte utilizando Microsoft Windows Server e Hyper-V.

O ambiente foi projetado para representar uma rede corporativa isolada, com serviços centralizados de autenticação, resolução de nomes e distribuição automática de endereços IP.

A infraestrutura está sendo construída de forma incremental, permitindo implementar, testar, documentar e validar cada serviço antes da próxima etapa.

### Objetivos da arquitetura

* Centralizar o gerenciamento de usuários e computadores;
* Utilizar o Active Directory Domain Services (AD DS) para autenticação e organização do ambiente;
* Utilizar DNS para resolução de nomes internos;
* Utilizar DHCP para distribuição automática de endereços IP;
* Implementar posteriormente compartilhamentos de arquivos e permissões;
* Aplicar políticas de domínio por meio de Group Policy Objects (GPOs);
* Automatizar tarefas administrativas utilizando PowerShell;
* Criar cenários de troubleshooting para simular problemas comuns de infraestrutura.

---

## 2. Ambiente de virtualização

O laboratório utiliza o **Hyper-V** como plataforma de virtualização.

As máquinas virtuais do laboratório são mantidas separadas do diretório do projeto GitHub para evitar que arquivos grandes, como discos virtuais e imagens de instalação, sejam versionados.

### Estrutura atual

| Componente          | Configuração                            |
| ------------------- | --------------------------------------- |
| Hypervisor          | Microsoft Hyper-V                       |
| Switch virtual      | `LAB-SWITCH`                            |
| Tipo do switch      | Internal                                |
| Rede                | `192.168.10.0/24`                       |
| Servidor principal  | `SRV-DC01`                              |
| Sistema operacional | Windows Server 2022 Standard Evaluation |
| Interface gráfica   | Desktop Experience                      |

---

## 3. Topologia de rede

O laboratório utiliza uma rede virtual interna criada no Hyper-V.

O switch `LAB-SWITCH` é do tipo **Internal**, permitindo comunicação entre o sistema operacional hospedeiro e as máquinas virtuais conectadas ao switch.

A rede não possui conexão direta com a rede física ou com a Internet neste momento.

### Sub-rede

```text
Rede:        192.168.10.0/24
Máscara:     255.255.255.0
Gateway:     não configurado
```

A utilização de uma rede isolada evita que o DHCP do laboratório entre em conflito com o DHCP do roteador da rede doméstica.

### Distribuição planejada de endereços

| Faixa                             | Finalidade                           |
| --------------------------------- | ------------------------------------ |
| `192.168.10.1 – 192.168.10.99`    | Infraestrutura e endereços estáticos |
| `192.168.10.10`                   | Servidor `SRV-DC01`                  |
| `192.168.10.100 – 192.168.10.200` | Pool DHCP                            |
| `192.168.10.201 – 192.168.10.254` | Reserva para expansão                |

A divisão entre endereços estáticos e endereços distribuídos pelo DHCP facilita a identificação dos componentes da infraestrutura e deixa espaço para futuras expansões.

---

## 4. Servidor principal

O servidor principal do laboratório é denominado:

```text
SRV-DC01
```

Ele atua como controlador de domínio e concentra atualmente os principais serviços de infraestrutura da rede.

### Configuração de rede

| Parâmetro | Valor            |
| --------- | ---------------- |
| Hostname  | `SRV-DC01`       |
| IPv4      | `192.168.10.10`  |
| Máscara   | `255.255.255.0`  |
| Gateway   | Não configurado  |
| DNS       | `192.168.10.10`  |
| Domínio   | `ad.labtech.lab` |

O endereço IP do servidor é estático para garantir que os demais equipamentos possam localizar de forma consistente os serviços de domínio e DNS.

---

## 5. Active Directory Domain Services

O ambiente utiliza o **Active Directory Domain Services (AD DS)** para gerenciamento centralizado de identidade e recursos.

### Domínio

```text
ad.labtech.lab
```

O servidor `SRV-DC01` atua como controlador de domínio.

A estrutura lógica planejada para o Active Directory é:

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
    ├── GRP-TI
    ├── GRP-RH
    └── GRP-FINANCEIRO
```

A estrutura organizacional será utilizada posteriormente para facilitar a aplicação de políticas, gerenciamento de usuários e controle de acesso aos recursos da rede.

---

## 6. DNS

O DNS é fornecido pelo próprio `SRV-DC01`.

```text
DNS Server:
192.168.10.10
```

O DNS é fundamental para o funcionamento do Active Directory, pois permite que os computadores encontrem os serviços e recursos do domínio por meio de nomes.

O servidor está configurado para utilizar a si próprio como servidor DNS.

### Validação

A configuração foi validada utilizando ferramentas nativas do Windows Server, incluindo:

```powershell
nslookup ad.labtech.lab
```

e:

```powershell
dcdiag /test:dns
```

O diagnóstico `dcdiag /test:dns` foi executado com sucesso, incluindo os testes de conectividade, DNS do domínio e partições do Active Directory.

---

## 7. DHCP

O serviço DHCP foi instalado e configurado no `SRV-DC01`.

O objetivo do DHCP é distribuir automaticamente configurações de rede para os clientes do laboratório.

### Escopo

```text
Nome:       LABTECH-LAN
Rede:       192.168.10.0/24
Início:     192.168.10.100
Fim:        192.168.10.200
Máscara:    255.255.255.0
```

O escopo possui **101 endereços IPv4 disponíveis** para concessão:

```text
200 - 100 + 1 = 101
```

### Opções DHCP

Os clientes recebem:

```text
Domínio DNS:
ad.labtech.lab

Servidor DNS:
192.168.10.10
```

Não foi configurado gateway padrão porque a rede do laboratório permanece isolada e não possui um roteador conectado ao `LAB-SWITCH`.

O WINS também não foi configurado, pois não é necessário para a arquitetura atual baseada em Active Directory e DNS.

### Duração da concessão

A duração padrão da concessão DHCP permanece configurada em:

```text
8 dias
```

### Validação

O escopo foi validado por meio do PowerShell:

```powershell
Get-DhcpServerv4Scope
```

As opções DHCP foram verificadas com:

```powershell
Get-DhcpServerv4OptionValue -ScopeId 192.168.10.0
```

A validação confirmou:

* escopo ativo;
* faixa `192.168.10.100–192.168.10.200`;
* máscara `/24`;
* domínio DNS `ad.labtech.lab`;
* servidor DNS `192.168.10.10`;
* duração de concessão de 8 dias.

Essa validação foi feita no próprio servidor. O funcionamento de ponta a ponta, com um cliente recebendo endereço e opções, ainda será testado quando o CLIENT-01 for criado.

---

## 8. Gateway e conectividade externa

O laboratório atualmente **não possui gateway padrão configurado**.

Isso é intencional.

A rede foi criada como um ambiente isolado para permitir testes de infraestrutura sem interferir na rede doméstica utilizada pelo computador hospedeiro.

Consequentemente, as máquinas conectadas exclusivamente ao `LAB-SWITCH` podem se comunicar entre si e com o host, mas não possuem acesso direto à Internet por meio dessa rede.

Caso seja necessário adicionar conectividade externa futuramente, será necessário projetar essa alteração de forma que o DHCP do laboratório não entre em conflito com o DHCP da rede física.

---

## 9. Serviços implementados

Até o momento, a infraestrutura possui os seguintes serviços configurados:

| Serviço                               | Servidor   | Status       |
| ------------------------------------- | ---------- | ------------ |
| Active Directory Domain Services      | `SRV-DC01` | Implementado |
| DNS                                   | `SRV-DC01` | Implementado |
| DHCP                                  | `SRV-DC01` | Implementado |(teste com cliente pendente)
| File Server                           | —          | Planejado    |
| GPO                                   | —          | Planejado    |
| Automação PowerShell                  | —          | Planejado    |
| Cliente Windows ingressado no domínio | —          | Planejado    |

---

## 10. Próximas etapas da arquitetura

A infraestrutura será expandida progressivamente.

### 10.1 File Server

Será implementado um servidor de arquivos utilizando compartilhamentos separados por departamento.

Estrutura planejada:

```text
D:\Shares\
├── TI\
├── RH\
├── Financeiro\
└── Publico\
```

O acesso será controlado por meio de grupos de segurança do Active Directory e permissões NTFS.

---

### 10.2 Group Policy

Serão criadas GPOs para demonstrar gerenciamento centralizado dos computadores do domínio.

Entre as configurações planejadas estão:

* mapeamento de unidades de rede;
* políticas de segurança;
* configurações de ambiente;
* aplicação de configurações por Unidade Organizacional (OU).

Mapeamento planejado:

```text
TI          → T:
RH          → R:
Financeiro  → F:
```

---

### 10.3 Cliente Windows

Será criada uma máquina virtual cliente conectada ao `LAB-SWITCH`.

O cliente deverá:

1. receber endereço IP via DHCP;
2. receber o servidor DNS `192.168.10.10`;
3. resolver o domínio `ad.labtech.lab`;
4. ingressar no domínio;
5. receber políticas de grupo;
6. acessar os compartilhamentos conforme as permissões configuradas.

---

### 10.4 Automação PowerShell

Será desenvolvido um conjunto de scripts para automatizar tarefas administrativas.

Um dos principais scripts planejados será responsável pela criação de usuários do domínio a partir de um arquivo CSV.

Exemplo conceitual:

```text
usuarios.csv
     │
     ▼
create-users.ps1
     │
     ├── Verificar usuário existente
     ├── Criar conta
     ├── Definir atributos
     ├── Adicionar aos grupos
     └── Registrar operações
```

---

## 11. Estratégia de validação

Cada serviço implementado será validado antes da continuidade do projeto.

A validação utilizará ferramentas nativas do Windows Server e testes funcionais.

Exemplos:

### Rede

```powershell
ipconfig
```

### DNS

```powershell
nslookup ad.labtech.lab
```

```powershell
Resolve-DnsName ad.labtech.lab
```

### Diagnóstico do Active Directory

```powershell
dcdiag /test:dns
```

### DHCP

```powershell
Get-DhcpServerv4Scope
```

```powershell
Get-DhcpServerv4OptionValue -ScopeId 192.168.10.0
```

### Conectividade

```powershell
Test-NetConnection
```

A documentação do projeto deverá registrar tanto a configuração quanto os resultados das validações relevantes.

---

## 12. Princípios da arquitetura

A implementação segue alguns princípios para manter o laboratório próximo de um ambiente corporativo real:

* Separação entre endereços estáticos e DHCP;
* DNS integrado ao Active Directory;
* Gerenciamento centralizado por domínio;
* Organização de usuários e computadores por OUs;
* Controle de acesso baseado em grupos;
* Separação entre permissões de compartilhamento e NTFS;
* Automação de tarefas repetitivas com PowerShell;
* Validação técnica após cada implementação;
* Documentação das decisões de infraestrutura;
* Registro de evidências por screenshots;
* Versionamento da documentação e dos scripts com Git.

---

## 13. Estado atual

A infraestrutura base do laboratório está funcional.

### Implementado e validado

* Hyper-V;
* switch virtual interno `LAB-SWITCH`;
* Windows Server 2022;
* hostname `SRV-DC01`;
* configuração IPv4 estática;
* Active Directory Domain Services;
* domínio `ad.labtech.lab`;
* DNS;
* diagnóstico DNS com `dcdiag`;
* DHCP (configuração verificada no servidor; teste com cliente pendente);
* escopo `LABTECH-LAN`;
* opções DHCP;
* documentação inicial da arquitetura.

### Planejado

* estrutura completa de OUs;
* grupos de segurança;
* usuários;
* File Server;
* permissões NTFS;
* compartilhamentos;
* GPOs;
* cliente Windows;
* automação PowerShell;
* cenários de troubleshooting;
* documentação final e evidências.

---

## 14. Diagrama lógico

A arquitetura atual pode ser representada da seguinte forma:

```text
                    HOST WINDOWS
                         │
                         │
                  Hyper-V / Internal
                         │
                    LAB-SWITCH
                         │
                         │
                  192.168.10.0/24
                         │
                         │
                 ┌───────┴───────┐
                 │               │
             SRV-DC01        CLIENT-01
           192.168.10.10     DHCP futuro
                 │
        ┌────────┼────────┐
        │        │        │
       AD DS    DNS      DHCP
        │        │        │
        └────────┴────────┘
                 │
          ad.labtech.lab
```

O `CLIENT-01` ainda será implementado nas próximas etapas do laboratório.

---

## 15. Observação sobre evolução

Esta documentação representa o estado da arquitetura durante a construção do laboratório e será atualizada conforme novos serviços forem implementados.

Alterações relevantes de rede, serviços, estrutura do Active Directory e políticas deverão ser registradas neste documento para manter a arquitetura documentada de acordo com o ambiente real.
