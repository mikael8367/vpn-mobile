# VPN Mobile - Minecraft LAN a Longa Distância

🎮 Jogue Minecraft LAN com amigos em qualquer lugar do mundo!

## O que é?

Sistema VPN simples baseado em WireGuard que conecta você e seus amigos numa rede privada virtual, permitindo jogar Minecraft LAN sem estar na mesma rede física.

## Como Funciona

1. **Servidor VPN** - Um dispositivo com IP fixo (seu PC, servidor cloud, etc)
2. **Clientes** - Seu celular, PC, console com a configuração WireGuard
3. **Rede Virtual** - Todos conectados como se estivessem na mesma LAN

## Requisitos

- WireGuard instalado (Android, iOS, Windows, Mac, Linux)
- Um servidor com IP público (AWS, DigitalOcean, Linode, etc)
- 5-10 minutos para configuração

## Instalação Rápida

### 1. Servidor VPN (Linux)

```bash
# Instalar WireGuard
sudo apt update
sudo apt install wireguard wireguard-tools

# Gerar chaves
cd /etc/wireguard
umask 077
wg genkey | tee privatekey | wg pubkey > publickey

# Ver suas chaves
cat privatekey
cat publickey
```

### 2. Configurar Interface

Crie `/etc/wireguard/wg0.conf`:

```ini
[Interface]
PrivateKey = [sua-chave-privada-do-servidor]
Address = 10.0.0.1/24
ListenPort = 51820
PostUp = iptables -A FORWARD -i %i -j ACCEPT; iptables -t nat -A POSTROUTING -o eth0 -j MASQUERADE
PostDown = iptables -D FORWARD -i %i -j ACCEPT; iptables -t nat -D POSTROUTING -o eth0 -j MASQUERADE
```

### 3. Adicionar Peers (Amigos)

Para cada amigo, gere chaves:

```bash
wg genkey | tee peer1_private | wg pubkey > peer1_public
```

Adicione ao `wg0.conf`:

```ini
[Peer]
PublicKey = [chave-publica-amigo]
AllowedIPs = 10.0.0.2/32
```

### 4. Ativar VPN

```bash
sudo systemctl enable wg-quick@wg0
sudo systemctl start wg-quick@wg0
wg show
```

### 5. Configurar no Celular/PC

1. Baixe WireGuard (Play Store, App Store)
2. Crie um arquivo `.conf` para cada cliente:

```ini
[Interface]
PrivateKey = [sua-chave-privada]
Address = 10.0.0.2/24
DNS = 8.8.8.8

[Peer]
PublicKey = [chave-publica-servidor]
Endpoint = seu-servidor.com:51820
AllowedIPs = 10.0.0.0/24
PersistentKeepalive = 25
```

3. Importe no WireGuard e conecte

## Testando

```bash
# Ping entre amigos
ping 10.0.0.2

# Abrir Minecraft
# Criar mundo LAN
# Amigos entram usando o IP 10.0.0.X da VPN
```

## Benchmarks

- Latência: ~10-50ms (dependendo localização)
- Bandwidth: Até 1Gbps
- CPU: Mínimo (~5-10% num servidor básico)
- Perfeito para Minecraft LAN

## Alternativas

- **Tailscale** - Mais fácil, mas paga pra múltiplos usuários
- **ZeroTier** - Sem servidor, mais simples
- **OpenVPN** - Mais complexo mas open-source

## Segurança

✅ Criptografia moderna (ChaCha20-Poly1305)  
✅ Sem logs  
✅ Sem rastreamento  
✅ Código aberto  

## Troubleshooting

**Não consegue conectar?**
- Verifique porta 51820 aberta no firewall
- Confirme IPs privados 10.0.0.x/24
- Teste com `wg show`

**Minecraft não encontra servidor?**
- Use IP 10.0.0.X do amigo
- Certifique que todos estão na mesma rede 10.0.0.0/24

## Próximos Passos

- [ ] Script de setup automático
- [ ] Interface gráfica para adicionar peers
- [ ] Suporte a múltiplas redes VPN
- [ ] App Android/iOS nativa

---

**Pronto para jogar!** 🎮