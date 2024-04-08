# Firewall_IPTABLES-

## Limpa todas as regras existentes
iptables -F iptables -X

## Estabelece política padrão DROP para INPUT e FORWARD
iptables -P INPUT DROP iptables -P FORWARD DROP

## Permiti o tráfego para interface de loopback
iptables -A INPUT -i lo -j ACCEPT

## Permiti o tráfego de ICMP echo-request limitado a 5 pacotes por segundo
iptables -A INPUT -p icmp --icmp-type echo-request -m limit --limit 5/second -j ACCEPT

## Permiti o acesso a HTTP (porta 80) e HTTPS (porta 443) para usuários internos e realizar NAT
iptables -A FORWARD -i ETH1 -o ETH0 -p tcp --dport 80 -j ACCEPT iptables -A FORWARD -i ETH1 -o ETH0 -p tcp --dport 443 -j ACCEPT iptables -t nat -A POSTROUTING -o ETH0 -j MASQUERADE

## Bloqueia o acesso a sites que contenham a palavra "games" e faz LOG
iptables -A FORWARD -m string --string "games" --algo kmp -j LOG --log-prefix "Blocked Games: " iptables -A FORWARD -m string --string "games" --algo kmp -j DROP

## Bloqueia o acesso ao site www.jogosonline.com.br, exceto para o IP do chefe
iptables -A FORWARD -d www.jogosonline.com.br -j DROP iptables -A FORWARD -d www.jogosonline.com.br -s 10.1.1.100 -j ACCEPT

## Permiti consultas DNS externas para rede interna e DMZ
iptables -A FORWARD -i ETH1 -p udp --dport 53 -j ACCEPT iptables -A FORWARD -i DMZ_INTERFACE -p udp --dport 53 -j ACCEPT

## Permiti o tráfego TCP destinado à máquina na DMZ (192.168.1.100) na porta 80
iptables -A FORWARD -d 192.168.1.100 -p tcp --dport 80 -j ACCEPT

## Redireciona os pacotes TCP destinados ao IP 200.20.5.1 porta 80 para 192.168.1.100 na DMZ
iptables -t nat -A PREROUTING -d 200.20.5.1 -p tcp --dport 80 -j DNAT --to-destination 192.168.1.100:80

## Permiti que a máquina na DMZ (192.168.1.100) responda pacotes TCP na porta 80 corretamente
iptables -A FORWARD -d 192.168.1.100 -p tcp --dport 80 -m state --state ESTABLISHED,RELATED -j ACCEPT
