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

# Quantos pacotes foram descartados pela regra?

 <img width="1363" height="668" alt="1" src="https://github.com/user-attachments/assets/0a9afb52-7751-46de-bd3e-06c84a044d40" />

 
Gravei vídeos de todo o processo — desde a criação da regra, o salvamento e a verificação dos logs — para documentar o comportamento em tempo real.
Este laboratório me ajudou a compreender melhor o fluxo de pacotes, a função da chain INPUT e como o `iptables` interage com o kernel para registrar eventos.

O **IptablesLogLab** é o primeiro passo de uma série de pequenos laboratórios que pretendo criar e publicar no GitHub, com o objetivo de estudar **segurança de redes, firewall, Linux e análise de tráfego** de forma prática e reprodutível.



