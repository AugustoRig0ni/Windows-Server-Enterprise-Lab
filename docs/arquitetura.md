# Arquitetura da Infraestrutura

## 1. Visão geral

O laboratório simula uma infraestrutura corporativa de pequeno porte (empresa fictícia **LABTECH**) com Windows Server 2022 e Hyper-V, em uma rede virtual isolada.

Um único servidor, `SRV-DC01`, concentra os serviços de identidade, nomes, endereçamento e arquivos. Um cliente Windows ingressado no domínio é usado para validar o ambiente.

Este documento é a visão geral. Os detalhes de cada serviço estão nos documentos específicos, listados na seção 5.

---

## 2. Virtualização

O ambiente usa **Microsoft Hyper-V**. As máquinas virtuais ficam fora da pasta do projeto, para que discos e imagens de instalação não sejam versionados no GitHub.

| Máquina | Sistema | Função |
| ------- | ------- | ------ |
| `SRV-DC01` | Windows Server 2022 Standard Evaluation (Desktop Experience) | AD DS, DNS, DHCP e File Server |
| `CLIENT-01` | Windows cliente | Estação do domínio usada nos testes |

### SRV-DC01

| Item | Configuração |
| ---- | ------------ |
| Memória | 4 GB |
| Disco principal | VHDX de 60 GB (volume `C:`) |
| Disco de backup | VHDX dinâmico de aproximadamente 40 GB, volume `E:` (rótulo `BACKUP`, NTFS), usado exclusivamente para backups |
| Rede | `192.168.10.10/24`, sem gateway, DNS `192.168.10.10` |

### CLIENT-01

| Item | Configuração |
| ---- | ------------ |
| Hostname | `DESKTOP-81PP8GF` (identificado como `CLIENT-01` neste projeto) |
| Endereço | `192.168.10.101`, obtido por DHCP |
| Domínio | `ad.labtech.lab` |
| Objeto no AD | Movido para a OU `Computadores` do laboratório |

---

## 3. Rede

O switch virtual `LAB-SWITCH` é do tipo **Internal**: permite a comunicação entre as VMs e o host, sem conexão com a rede física ou com a Internet.

```text
Rede:        192.168.10.0/24
Máscara:     255.255.255.0
Gateway:     não configurado
```

A rede isolada evita conflito entre o DHCP do laboratório e o do roteador da rede doméstica.

| Faixa | Finalidade |
| ----- | ---------- |
| `192.168.10.1 – 192.168.10.99` | Infraestrutura e endereços estáticos (`SRV-DC01` = `.10`) |
| `192.168.10.100 – 192.168.10.200` | Pool DHCP |
| `192.168.10.201 – 192.168.10.254` | Faixa livre para expansão |

---

## 4. Diagrama lógico

```text
                    HOST WINDOWS
                         │
                  Hyper-V / Internal
                         │
                    LAB-SWITCH
                  192.168.10.0/24
                         │
                 ┌───────┴───────┐
                 │               │
             SRV-DC01        CLIENT-01
           192.168.10.10    192.168.10.101
                 │
     ┌─────┬─────┼──────┐
   AD DS  DNS  DHCP  File Server
                 │
          ad.labtech.lab
```

---

## 5. Serviços implementados

| Serviço | Servidor | Documento |
| ------- | -------- | --------- |
| Active Directory Domain Services | `SRV-DC01` | [active-directory.md](active-directory.md) |
| DNS | `SRV-DC01` | [dns-dhcp.md](dns-dhcp.md) |
| DHCP | `SRV-DC01` | [dns-dhcp.md](dns-dhcp.md) |
| File Server (SMB + NTFS) | `SRV-DC01` | [file-server.md](file-server.md) |
| Group Policy | `SRV-DC01` | [gpo.md](gpo.md) |
| Windows Server Backup | `SRV-DC01` | [backup-e-recuperacao.md](backup-e-recuperacao.md) |

Resultados dos testes: [testes-e-validacoes.md](testes-e-validacoes.md).

---

## 6. Limitações da arquitetura

- **Ponto único:** um servidor concentra todas as funções. É uma simplificação de laboratório (ver decisão D04 em [decisoes.md](decisoes.md)).
- **Backup no mesmo host:** o VHDX de backup está no computador físico que hospeda a VM.
- **Sem Internet:** a rede isolada impede, por exemplo, o uso do Windows Update.
- **Licença de avaliação:** Windows Server Evaluation, instalado em 30/09/2026 (180 dias).

Não fazem parte desta versão: firewall dedicado, segmentação de rede, monitoramento centralizado e automação de usuários (ver [evolucoes-futuras.md](evolucoes-futuras.md)).

---

## 7. Princípios seguidos

- Separação entre endereços estáticos e DHCP;
- DNS integrado ao Active Directory;
- Gerenciamento centralizado por domínio;
- Organização de usuários e computadores por OUs;
- Controle de acesso baseado em grupos;
- Separação entre permissões de compartilhamento (SMB) e NTFS;
- Validação técnica após cada etapa;
- Registro das decisões de infraestrutura;
- Versionamento da documentação com Git.

---

## 8. Observação sobre evolução

Este documento descreve a versão 1 do laboratório. O histórico de versões anteriores (incluindo o estado em que o cliente ainda não existia) está no histórico de commits do repositório.
