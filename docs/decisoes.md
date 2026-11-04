# Decisões de Projeto

Aqui ficam as escolhas que fiz no laboratório e o motivo de cada uma. Serve para eu lembrar por que fiz assim e para conseguir explicar o projeto depois.

---

## D01 — Rede isolada (Internal Switch), sem gateway

Usei o switch `LAB-SWITCH` do tipo Internal, com a rede `192.168.10.0/24` e sem gateway padrão.

- **Motivo:** evitar conflito entre o DHCP do laboratório e o DHCP do roteador de casa, e não mexer na rede do computador hospedeiro.
- **Consequência:** as VMs conversam entre si e com o host, mas não têm Internet (nada de Windows Update ou download de ferramentas).
- **Se eu precisar de Internet depois:** adicionar uma segunda placa de rede/NAT ao servidor e configurar forwarders no DNS, sem deixar o DHCP do lab vazar para a rede física.

## D02 — IP estático no servidor

O `SRV-DC01` usa `192.168.10.10/24`, com o DNS apontando para ele mesmo.

- **Motivo:** o controlador de domínio precisa ser encontrado sempre no mesmo endereço, porque AD e DNS dependem disso.

## D03 — Faixa estática separada da faixa DHCP

`.1` a `.99` para infraestrutura, `.100` a `.200` para DHCP (101 endereços) e `.201` a `.254` livres para expansão.

- **Motivo:** fica fácil identificar o que é infraestrutura e sobra espaço para crescer.

## D04 — DC, DHCP e File Server no mesmo servidor

Concentrei AD DS, DNS, DHCP e File Server no `SRV-DC01`.

- **Alternativa:** servidores separados, por exemplo um `SRV-FS01` só para arquivos.
- **Motivo:** simplificar o laboratório, com menos VMs e menos consumo de hardware.
- **Em produção:** o File Server ficaria fora do DC, para reduzir a superfície de ataque e isolar falhas.

## D05 — Domínio `ad.labtech.lab`

- **Alternativas:** `.internal` (reservado para uso interno) ou um subdomínio de um domínio público.
- **Motivo:** deixa claro que é um laboratório e não depende de domínio público.
- **Trade-off:** `.lab` não é oficialmente reservado para uso privado. Como a rede é virtual e isolada, não tem impacto prático, e não valia a pena recriar o ambiente por isso.

## D06 — Sem WINS

- **Motivo:** não preciso dele numa arquitetura baseada em Active Directory e DNS.

## D07 — Concessão DHCP mantida em 8 dias

Deixei o padrão do Windows Server (confirmado no ambiente: `8.00:00:00`).

- **Motivo:** o laboratório tem poucos clientes e não precisa renovar concessões com frequência.
- **Observação:** em redes com muita troca de dispositivos, concessões menores são comuns.

## D08 — Windows Server com Desktop Experience

- **Alternativa:** Server Core.
- **Motivo:** facilitar o aprendizado com a interface gráfica visível, para observar problemas que podem acontecer e praticar procedimentos cotidianos de configuração e ajustes.

## D09 — VMs, discos e ISOs fora do repositório

Mantenho as VMs separadas da pasta do projeto e ignoro `.vhdx`, `.iso` e similares no `.gitignore`.

- **Motivo:** não versionar arquivos grandes. O repositório guarda só documentação, scripts e evidências.

## D10 — Windows Server Evaluation

- **Motivo:** uso de estudo, sem custo de licença.
- **Instalado em:** 30/09/2026, 10:17:48 (`(Get-CimInstance Win32_OperatingSystem).InstallDate`).
- **Prazo estimado:** cerca de 29/03/2027. O tempo exato dá para conferir com `slmgr /dlv`.

## D11 — Disco virtual dedicado ao backup

- **Decisão:** criar um segundo VHDX dinâmico, de cerca de 40 GB, usado exclusivamente para backups (`E:`, rótulo `BACKUP`).
- **Limitação:** o VHDX está no mesmo computador físico que hospeda a VM. Serve para o laboratório, mas não protege contra a perda do host. Em produção, o backup iria para um local externo.

## D12 — Permissões SMB amplas e NTFS restritivas

- **Decisão:** no compartilhamento (SMB), `Authenticated Users` com Change e `Administrators` com Full Control. No NTFS, o grupo de cada departamento com Modify.
- **Motivo:** a camada SMB oferece acesso à pasta compartilhada e as permissões NTFS controlam, na prática, o acesso por departamento. Assim a regra por grupo fica em um só lugar.

## D13 — Mapeamento de unidades por GPO com Item-Level Targeting

- **Decisão:** uma GPO de Preferences (`GPO-Mapeamento-Unidades`) mapeia `P:` para todos e `T:`, `R:` ou `F:` conforme o grupo do usuário.
- **Resultado:** cada usuário recebe o `P:` e só a unidade do seu departamento, sem uma GPO por setor.

## D14 — Compartilhamentos em `C:\Compartilhamentos`

- **Decisão:** as pastas dos compartilhamentos (`TI`, `RH`, `Financeiro` e `Publico`) ficam em `C:\Compartilhamentos`.
- **Observação:** o planejamento inicial previa `D:\Shares`.
- **Motivo:** preferência de organização durante a implementação. Não houve uma justificativa técnica específica, nem limitação de armazenamento, para usar o volume `C:` em vez de outro caminho.
