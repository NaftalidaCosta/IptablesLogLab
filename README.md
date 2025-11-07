# IptablesLogLab
Criei um laboratório no VirtualBox chamado NetLogLab para testar e demonstrar como registrar tráfego HTTP (porta 80) com iptables e persistir as regras com netfilter-persistent. Gravei os comandos, validei os logs via dmesg e documentei todo o processo para estudo e partilha.


**IptablesLogLab — HTTP Logging Lab (iptables + VirtualBox)**

---

Criei o **iptables**, um laboratório virtual no **Vmware Worstation** que desenvolvi para estudar e demonstrar como funcionam as regras de logging no **iptables** e a persistência de configurações com o **netfilter-persistent**.




https://github.com/user-attachments/assets/d256cb17-6e49-49e1-8bd4-9c1a76cde7ec





Na fase inicial do laboratório, listei as regras do `iptables` e confirmei que não havia nenhuma regra nas chains **INPUT** e **OUTPUT**. Depois acessei o site de teste público `testphp.vulnweb.com` pela porta TCP **80** e o conteúdo foi carregado normalmente — isso serviu como base para validar a conectividade antes da configuração do firewall.

Em seguida, adicionei a seguinte regra de log:

```bash
iptables --table filter -A INPUT -i ens33 -p tcp --sport 80 -d 172.22.12.0/24 -j LOG --log-prefix "HTTP INCOMING: " --log-level 4
```

O objetivo dessa regra foi registrar todo tráfego HTTP (porta 80) vindo de servidores externos e destinado à minha rede local. Usei o nível de log `4` (warn) para garantir que as mensagens fossem capturadas no `dmesg`.

Depois disso, executei a sequência de comandos para salvar e persistir as regras:



https://github.com/user-attachments/assets/1fe2d483-734e-4d87-b8ca-eb3fe5c78371
>
*PARTE-2*

https://github.com/user-attachments/assets/5b9f8d51-e86c-434d-8d68-8b1e04ab8506




```bash
netfilter-persistent save  ---> salva a regra no netfilter de forma persistente.
netfilter-persistent reload ---> Recarrega as regras salvas.
iptables-save ---> salva as regras no iptables
iptables-save > /etc/iptables/rules.v4 ---> salva as regras no arquivo rules.v4 (IPv4) localizado dentro da pasta iptables. 
service netfilter-persistent restart ---> reinicia o serviço "netfilter-persistent.service"
```

Reiniciei o navegador e acessei novamente o mesmo site de teste. Em seguida, usei o comando:
# 


https://github.com/user-attachments/assets/bacb399b-811a-473c-8e2a-7937ff3c2920



```bash
dmesg -SHT -l 4 | grep "HTTP" | tail -n 2 ---> Mostra todas as linhas de logs de nivel 4 (warn), que contém a palavra "HTTP" de forma legível para Humanos 
```

Os logs apareceram com sucesso, confirmando que a regra estava a funcionar corretamente.


`` $sudo dmesg -SHT -l 4 | grep "HTTP" | tail -n 2 `` ---> comando 

```
[sudo] password for offsec: 
[qui nov  6 23:30:59 2025] HTTP INCOMING: IN=ens33 OUT= MAC=b3:0c:00:be:59:b4:44:f6:p9:8d:8d:20:01:00 SRC=44.228.249.3 DST=172.22.12.8 LEN=60 TOS=0x00 PREC=0x00 TTL=48 ID=0 DF PROTO=TCP SPT=80 DPT=35726 WINDOW=62643 RES=0x00 ACK SYN URGP=0 
[qui nov  6 23:31:00 2025] HTTP INCOMING: IN=ens33 OUT= MAC=b3:0c:00:b0:59:b4:08:f6:p9:8d:d4:20:01:00 SRC=44.228.249.3 DST=172.22.12.8 LEN=60 TOS=0x00 PREC=0x00 TTL=47 ID=0 DF PROTO=TCP SPT=80 DPT=35736 WINDOW=62643 RES=0x00 ACK SYN URGP=0  
``` 

Os dos últimos logs da regra difinida anteriormente foram registros no dia 6 de novembro as 23 horas e 32 minutos.
O pacote contém informações da camada 2 e 3 do modelo OSI. O frame foi recebido pela interface "ens33", o endereço IP de origem é público (Servidor Web) e o IP de destino é um endereço privado (LAN), as bandeiras ou as flags difinidas são "SYN-ACK". Quando eu tentei acessar o domínio alvo a minha máquina enviou um pacote com a flag SYN com destino a porta 80. O servidor respondeu a primeira flag do 3-way Handshake com a porta de destino "35786", mas quando o pacote chegou na Network Interface Card (NIC) ou interface de rede foi dropado pela regra. 
# Quantos pacotes foram descartados pela regra?

 <img width="1363" height="668" alt="1" src="https://github.com/user-attachments/assets/0a9afb52-7751-46de-bd3e-06c84a044d40" />
 140 pacotes foram descartados pela regra.
 
# Teste Final

<img width="1363" height="671" alt="image" src="https://github.com/user-attachments/assets/fdd932c6-e469-4aa3-a272-bce6146df13d" />


As 23 horas do dia 6, tentei acessar pela teceira vezes o site alvo e não foi possivel terminar o 3-way handshake e processar o counteúdo do site com êxito.
 
Gravei vídeos e tirei fotos de todo o processo — desde a criação da regra, o salvamento e a verificação dos logs — para documentar o comportamento em tempo real.
Este laboratório me ajudou a compreender melhor o fluxo de pacotes, a função da chain INPUT e como o `iptables` interage com o kernel para registrar eventos.

O **IptablesLogLab** é o primeiro passo de uma série de pequenos laboratórios que pretendo criar e publicar no GitHub, com o objetivo de estudar **segurança de redes, firewall, Linux e análise de tráfego** de forma prática e reprodutível.



