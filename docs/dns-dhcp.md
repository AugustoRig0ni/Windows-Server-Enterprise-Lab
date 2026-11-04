# DNS e DHCP

Os dois serviços rodam no `SRV-DC01`.

## DNS

| Item | Valor |
| ---- | ----- |
| Servidor DNS | `192.168.10.10` |
| Domínio DNS | `ad.labtech.lab` |
| Cliente do DNS no servidor | O próprio `SRV-DC01`, com o seu endereço |

O DNS permite que os computadores localizem os serviços do domínio. Sem ele, o AD DS não funciona.

### Validação

```powershell
nslookup ad.labtech.lab
dcdiag /test:dns
```

O diagnóstico `dcdiag /test:dns` foi executado com sucesso, e a resolução de nomes foi testada. Do cliente, a resolução de `ad.labtech.lab` retornou `192.168.10.10` e a porta TCP 53 do servidor DNS respondeu.

## DHCP

### Escopo

| Item | Valor |
| ---- | ----- |
| Nome | `LABTECH-LAN` |
| Rede | `192.168.10.0/24` |
| Faixa distribuída | `192.168.10.100` a `192.168.10.200` (101 endereços) |
| Duração da concessão | 8 dias (`8.00:00:00`), padrão do Windows Server |
| Gateway | Não configurado |

### Opções distribuídas

| Opção | Valor |
| ----- | ----- |
| Servidor DNS | `192.168.10.10` |
| Sufixo DNS | `ad.labtech.lab` |

Não há gateway porque a rede é isolada e não tem roteador conectado ao `LAB-SWITCH`. O WINS não foi configurado, pois não é necessário em uma arquitetura baseada em AD e DNS.

### Autorização no Active Directory

O servidor DHCP foi **autorizado no Active Directory**, e o escopo está ativo.

### Validação no servidor

```powershell
Get-DhcpServerv4Scope
Get-DhcpServerv4OptionValue -ScopeId 192.168.10.0
Get-DhcpServerv4Lease -ScopeId 192.168.10.0
```

Confirmaram: escopo ativo, faixa e máscara corretas, DNS e sufixo distribuídos, e a concessão do cliente.

### Validação com o cliente

- O `CLIENT-01` recebeu `192.168.10.101` por DHCP.
- Recebeu `192.168.10.10` como servidor DNS.
- A configuração foi conferida no cliente com `ipconfig /all`.

Após a reinicialização do servidor e do cliente, o DNS e o DHCP apareceram saudáveis no Server Manager.
