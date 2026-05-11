# NetPractice — Guia Completo de Teoria e Resolução

> **Objetivo:** Dominar conceitos de redes TCP/IP e resolver todos os 10 níveis do NetPractice com confiança.

---

## ÍNDICE
1. [Conceitos Fundamentais](#conceitos-fundamentais)
2. [Endereçamento IPv4](#endereçamento-ipv4)
3. [Máscaras e CIDR](#máscaras-e-cidr)
4. [Cálculo de Redes](#cálculo-de-redes)
5. [Todos os Cálculos Essenciais](#todos-os-cálculos-essenciais)
6. [Roteamento e Gateways](#roteamento-e-gateways)
7. [Método de Resolução (Checklist)](#método-de-resolução-checklist)
8. [Frases para Avaliação](#frases-para-avaliação)
9. [Exemplos Resolvidos](#exemplos-resolvidos)

---

## Conceitos Fundamentais

### O que é uma rede?
Uma **rede** é um conjunto de dispositivos (PCs, servidores, roteadores) conectados e capazes de se comunicar.

- **Comunicação direta:** quando dois hosts estão na **mesma rede**, falam direto (sem intermediário).
- **Comunicação indireta:** quando dois hosts estão em **redes diferentes**, precisam de um **roteador** (gateway) para se comunicar.

### Por que precisa de máscara?
O IP sozinho não diz se dois computadores estão na mesma rede. A **máscara** nos diz qual parte do IP identifica a rede e qual parte identifica o host dentro da rede.

**Exemplo:**
- IP: `192.168.1.5` com máscara `255.255.255.0`
  - Parte de rede: `192.168.1` (identificada pela máscara)
  - Parte de host: `5` (o computador específico)
- IP: `192.168.2.5` com máscara `255.255.255.0`
  - Parte de rede: `192.168.2` (diferente!)
  - Parte de host: `5`

**Resultado:** mesmo com o mesmo último octeto, estão em redes diferentes.

### Fato importante: AND binário
Para saber qual é a **rede** de um IP:
```
Rede = IP AND Máscara
```

O `AND` funciona assim (bit por bit):
```
1 AND 1 = 1
1 AND 0 = 0
0 AND 1 = 0
0 AND 0 = 0
```

**Exemplo prático:**
```
IP:     192.168.1.130
Máscara: 255.255.255.192

Último octeto em binário:
IP:     10000010 (130)
Máscara: 11000000 (192)
---
Rede:   10000000 (128)

Resultado: Rede = 192.168.1.128
```

---

## Endereçamento IPv4

### Estrutura do IPv4
Um endereço IPv4 tem **4 octetos** (4 números de 0 a 255), separados por ponto.

```
a . b . c . d
0-255 . 0-255 . 0-255 . 0-255
```

**Exemplo válido:** `192.168.1.5`
**Exemplos inválidos:** `256.168.1.5` (256 > 255), `192.168.1.-1` (negativo)

### Classes Históricas (contexto, menos usado hoje)
No modelo antigo (classful), o **primeiro octeto** definia a classe e a máscara "padrão":

- **Classe A:** `1.0.0.0` a `126.255.255.255` (máscara padrão `/8` ou `255.0.0.0`)
- **Classe B:** `128.0.0.0` a `191.255.255.255` (máscara padrão `/16` ou `255.255.0.0`)
- **Classe C:** `192.0.0.0` a `223.255.255.255` (máscara padrão `/24` ou `255.255.255.0`)
- **Classe D:** `224.0.0.0` a `239.255.255.255` (multicast, não é rede de host comum)
- **Classe E:** `240.0.0.0` a `255.255.255.255` (reservada/experimental)

Por que isso é "histórico":

- Com classes fixas (/8, /16, /24), havia muito desperdício de endereços.
- Hoje usamos **CIDR** (ex.: `/19`, `/27`, `/30`) para ajustar o tamanho da rede conforme necessidade.
- Na prática do NetPractice, você sempre decide conectividade com **IP + máscara atual**, não apenas pela "classe" do IP.

### Faixas Privadas (usadas em LANs)
As faixas privadas (RFC 1918) são usadas em redes internas:

- `10.0.0.0/8` (10.0.0.0 a 10.255.255.255)
- `172.16.0.0/12` (172.16.0.0 a 172.31.255.255)
- `192.168.0.0/16` (192.168.0.0 a 192.168.255.255)

Regras importantes:

- Esses IPs **não são roteáveis na internet pública**.
- Para sair para a internet, normalmente o roteador aplica **NAT** (tradução privado -> público).
- Em LAN, funcionam normalmente; ainda precisa configurar máscara/gateway/rotas corretamente.

Fora dessas faixas (e das reservadas especiais), os IPs são tratados como **públicos**.

### Faixas e Endereços Especiais (importante para avaliação)
- `127.0.0.0/8`: loopback (`127.0.0.1` = localhost, não sai para rede física)
- `169.254.0.0/16`: link-local (APIPA, autoconfiguração quando não há DHCP)
- `0.0.0.0`:
  - como IP de host: inválido
  - como rota `0.0.0.0/0`: rota padrão (default route)
- `255.255.255.255`: broadcast limitado (mesmo enlace)

Observação prática para prova: em NetPractice, normalmente você só precisa evitar IP inválido, rede, broadcast e duplicado. Mas conhecer essas faixas mostra maturidade técnica na explicação.

---

## Máscaras e CIDR

### O que é CIDR?
**CIDR** (Classless Inter-Domain Routing) é a notação `/número` que diz quantos bits da esquerda (começando do primeiro octeto) são de rede.

```
/24 = 24 bits de rede, 32-24=8 bits de host
/25 = 25 bits de rede, 32-25=7 bits de host
/26 = 26 bits de rede, 32-26=6 bits de host
```

### Conversão: CIDR ↔ Máscara

| CIDR | Máscara | Bits de host | Hosts por rede |
|------|---------|---|---|
| /8 | 255.0.0.0 | 24 | ~16 milhões |
| /16 | 255.255.0.0 | 16 | ~65 mil |
| /24 | 255.255.255.0 | 8 | 254 |
| /25 | 255.255.255.128 | 7 | 126 |
| /26 | 255.255.255.192 | 6 | 62 |
| /27 | 255.255.255.224 | 5 | 30 |
| /28 | 255.255.255.240 | 4 | 14 |
| /29 | 255.255.255.248 | 3 | 6 |
| /30 | 255.255.255.252 | 2 | 2 |
| /31 | 255.255.255.254 | 1 | 2 (ponto-a-ponto) |
| /32 | 255.255.255.255 | 0 | 1 (host único) |

### Como contar bits 1 na máscara

Cada octeto tem um valor em bits `1`:

| Decimal | Binário | Bits 1 |
|---------|---------|--------|
| 0 | 00000000 | 0 |
| 128 | 10000000 | 1 |
| 192 | 11000000 | 2 |
| 224 | 11100000 | 3 |
| 240 | 11110000 | 4 |
| 248 | 11111000 | 5 |
| 252 | 11111100 | 6 |
| 254 | 11111110 | 7 |
| 255 | 11111111 | 8 |

**Exemplo:**
```
Máscara: 255.255.255.192
Bits:    8   + 8   + 8   + 2   = 26
Prefixo: /26
```

### Truque rápido: Calcular o bloco (subnet step)

Para o octeto "quebrado" (o último que não é 255 ou 0):

```
Bloco = 256 - valor_do_octeto
```

Isso diz qual é o "salto" entre sub-redes.

**Exemplos:**
- `/24` (máscara `.0`): bloco = 256-0 = 256 (uma só rede)
- `/25` (máscara `.128`): bloco = 256-128 = 128 (duas sub-redes: 0-127, 128-255)
- `/26` (máscara `.192`): bloco = 256-192 = 64 (quatro sub-redes: 0-63, 64-127, 128-191, 192-255)
- `/27` (máscara `.224`): bloco = 256-224 = 32 (oito sub-redes)
- `/28` (máscara `.240`): bloco = 256-240 = 16 (16 sub-redes)
- `/29` (máscara `.248`): bloco = 256-248 = 8
- `/30` (máscara `.252`): bloco = 256-252 = 4 (quatro sub-redes)

---

## Cálculo de Redes

### Endereços Especiais em uma Rede

Para qualquer rede (ex.: `192.168.1.0/24`):

1. **Network Address (endereço de rede):** todos os bits de host = 0
   - **Não pode usar em host** (reservado para identificar a rede)
   - Exemplo: `192.168.1.0`

2. **Broadcast Address (endereço de broadcast):** todos os bits de host = 1
   - **Não pode usar em host** (reservado para enviar mensagem para todos na rede)
   - Exemplo: `192.168.1.255`

3. **Hosts válidos:** de (network + 1) até (broadcast - 1)
   - Exemplo: `192.168.1.1` até `192.168.1.254`

### Método Prático: Qual rede um IP pertence?

**Passo 1:** Identifique o octeto "quebrado" (o que não é 255 ou 0 na máscara)

**Passo 2:** Calcule bloco = 256 - valor do octeto

**Passo 3:** Divida o valor do IP naquele octeto pelo bloco (arredonde para baixo) e multiplique pelo bloco

**Exemplo 1: IP `192.168.1.130` com máscara `/26` (255.255.255.192)**
```
Octeto quebrado: o último (192)
Bloco: 256 - 192 = 64
Valor do octeto: 130

Dividir: 130 / 64 = 2.03... → arredondar para 2
Rede: 2 × 64 = 128

Rede: 192.168.1.128
Broadcast: 192.168.1.128 + 64 - 1 = 192.168.1.191
Hosts válidos: 192.168.1.129 até 192.168.1.190
```

**Exemplo 2: IP `10.42.0.5` com máscara `/16` (255.255.0.0)**
```
Octeto quebrado: o segundo (0)
Mas 0 significa: esse octeto e todos depois são de host.

Vamos olhar o terceiro octeto:
Octeto: 0, Máscara: 0 → bloco = 256 - 0 = 256 (todo o octeto é host)

Então a rede é determinada pelos dois primeiros octetos:
Rede: 10.42.0.0
Broadcast: 10.42.255.255
Hosts válidos: 10.42.0.1 até 10.42.255.254
```

**Exemplo 3: IP `172.16.50.20` com máscara `/25` (255.255.255.128)**
```
Octeto quebrado: o último (128)
Bloco: 256 - 128 = 128
Valor do octeto: 20

Dividir: 20 / 128 = 0.156... → arredondar para 0
Rede: 0 × 128 = 0

Rede: 172.16.50.0
Broadcast: 172.16.50.0 + 128 - 1 = 172.16.50.127
Hosts válidos: 172.16.50.1 até 172.16.50.126
```

### Verificar se dois IPs estão na mesma rede

```
Passo 1: Calcule a rede de ambos os IPs
Passo 2: Se redes = iguais → MESMA REDE (podem falar direto)
Passo 3: Se redes ≠ diferentes → REDES DIFERENTES (precisam roteador)
```

---

## Todos os Cálculos Essenciais

Esta seção lista **todos os cálculos** que você pode precisar ao resolver NetPractice, com explicação teórica do por quê.

### 1️⃣ Cálculo: Converter CIDR (ex.: /24) → Máscara em Decimal

**Por que fazer:**
- Às vezes a interface mostra apenas `/24`, mas você precisa do valor em decimal (`255.255.255.0`).
- Algumas pessoas acham mais fácil trabalhar com decimais; outras com CIDR.

**Como fazer:**
- CIDR = número de bits 1 contínuos na máscara.
- Cada octeto com 8 bits 1 vale 255.
- Octeto "quebrado" tem valor que corresponde aos bits 1 restantes.

**Fórmula rápida:**
```
/8  = 255.0.0.0
/16 = 255.255.0.0
/24 = 255.255.255.0
/25 = 255.255.255.128
/26 = 255.255.255.192
/27 = 255.255.255.224
/28 = 255.255.255.240
/29 = 255.255.255.248
/30 = 255.255.255.252
```

**Tabela octeto quebrado:**
```
/17 → /16 + 1 bit  → octeto 3 = 128 → 255.255.128.0
/18 → /16 + 2 bits → octeto 3 = 192 → 255.255.192.0
/19 → /16 + 3 bits → octeto 3 = 224 → 255.255.224.0
/20 → /16 + 4 bits → octeto 3 = 240 → 255.255.240.0
```

**Exemplo NetPractice (Level 1):**
```
Você vê na interface: "Mask: 255.255.255.0"
Você precisa saber: qual é o CIDR?
Solução: conte bits 1 = 8 + 8 + 8 + 0 = 24 → /24
```

---

### 2️⃣ Cálculo: Converter Máscara Decimal → CIDR (ex.: 255.255.255.192)

**Por que fazer:**
- Interface NetPractice às vezes mostra apenas em decimal.
- Você precisa saber o CIDR para usar em cálculos de rede.

**Como fazer:**
- Conte bits 1 em cada octeto.
- Use a tabela de octetos:
  - 255 = 8 bits 1
  - 254 = 7 bits 1
  - 252 = 6 bits 1
  - 248 = 5 bits 1
  - 240 = 4 bits 1
  - 224 = 3 bits 1
  - 192 = 2 bits 1
  - 128 = 1 bit 1
  - 0 = 0 bits 1

**Exemplo:**
```
Máscara: 255.255.255.192
Bits:    8 + 8 + 8 + 2 = 26
CIDR: /26
```

**Exemplo NetPractice (Level 2):**
```
Interface mostra: "Mask: 255.255.255.32"
Primeiro problema: 32 não é valor válido de octeto (máximo 255)!
→ Erro detectado: máscara inválida
(Só valores válidos: 255, 254, 252, 248, 240, 224, 192, 128, 0)
```

---

### 3️⃣ Cálculo: Calcular "Bloco" (subnet step) para o octeto quebrado

**Por que fazer:**
- O bloco determina o "salto" entre sub-redes.
- Sem isso, você não consegue calcular rede e broadcast rápido.

**Como fazer:**
```
Bloco = 256 - valor_do_octeto_quebrado
```

**Por que funciona:**
- Um octeto tem 256 valores possíveis (0–255).
- Se a máscara "consome" alguns valores para rede, o resto fica para host.
- O bloco é o tamanho de cada grupo.

**Exemplos:**
```
/24 (máscara 255.255.255.0)    → bloco = 256 - 0 = 256
/25 (máscara 255.255.255.128)  → bloco = 256 - 128 = 128
/26 (máscara 255.255.255.192)  → bloco = 256 - 192 = 64
/27 (máscara 255.255.255.224)  → bloco = 256 - 224 = 32
/28 (máscara 255.255.255.240)  → bloco = 256 - 240 = 16
/29 (máscara 255.255.255.248)  → bloco = 256 - 248 = 8
/30 (máscara 255.255.255.252)  → bloco = 256 - 252 = 4
```

**Exemplo NetPractice (Level 3+):**
```
Interface C1: IP 127.0.0.1 / Mask 255.255.255.252
Passo 1: máscara 252 (não é 255 nem 0) → octeto quebrado
Passo 2: bloco = 256 - 252 = 4
Resultado: redes saltam de 4 em 4
  Sub-redes: 127.0.0.0, 127.0.0.4, 127.0.0.8, 127.0.0.12, ...
```

---

### 4️⃣ Cálculo: Determinar Endereço de Rede (Network Address)

**Por que fazer:**
- Network address é o primeiro IP de uma sub-rede e é reservado (não pode usar em host).
- Precisas saber qual é para validar se um IP é rede ou para determinar a rede de um host.

**Como fazer (método do bloco):**

```
Passo 1: Identifique o octeto quebrado (aquele que não é 255 ou 0 na máscara)
Passo 2: Calcule bloco = 256 - máscara
Passo 3: Divida o valor do octeto IP pelo bloco (arredonde para baixo)
Passo 4: Multiplique o resultado pelo bloco
Resultado: esse é o valor do octeto de rede
```

**Exemplo 1:** IP `192.168.1.130` com máscara `/26` (255.255.255.192)
```
Octeto quebrado: último (192)
Bloco: 256 - 192 = 64

Valor do octeto IP: 130
Cálculo: 130 / 64 = 2.03... → arredondar para 2 (floor)
Rede: 2 × 64 = 128

Network address: 192.168.1.128
```

**Exemplo 2:** IP `10.20.50.5` com máscara `/24` (255.255.255.0)
```
Octeto quebrado: último (0)
Bloco: 256 - 0 = 256 (todo o octeto é host)

Então todos os IPs começados com 10.20.50.X estão na mesma rede.
Network address: 10.20.50.0
```

**Exemplo NetPractice (Level 1):**
```
A1: 104.93.23.364 com 255.255.255.0 ← inválido (octeto 364 > 255)
B1: 104.98.23.12 com 255.255.255.0 ← rede = 104.98.23.0

Problema: A1 e B1 não estão na mesma rede!
A1 deveria ser 104.98.23.X para combinar com B1

Verificação:
- B1 = 104.98.23.12 / 255.255.255.0
- Bloco: 256 - 0 = 256
- Rede: 104.98.23.0
- A1 deve ser corrigida para 104.98.23.13 (mesmo rede 104.98.23.0)
```

**Exemplo 3:** IP `172.16.50.200` com máscara `/16` (255.255.0.0)
```
Octeto quebrado: segundo (0)
Bloco: 256 - 0 = 256

Então a rede é determinada pelos primeiros 2 octetos.
Network address: 172.16.0.0
```

---

### 5️⃣ Cálculo: Determinar Endereço de Broadcast

**Por que fazer:**
- Broadcast address é o último IP de uma sub-rede e é reservado (não pode usar em host).
- Precisas saber qual é para validar se um IP é broadcast.

**Como fazer:**
```
Broadcast = Network + Bloco - 1
```

**Por que funciona:**
- Network é o primeiro IP válido.
- Bloco é o tamanho da sub-rede.
- Broadcast é o último IP desse range.

**Exemplo 1:** Network `192.168.1.128/26`
```
Network: 192.168.1.128
Bloco: 64
Broadcast: 128 + 64 - 1 = 192.168.1.191
```

**Exemplo 2:** Network `10.0.0.0/24`
```
Network: 10.0.0.0
Bloco: 256
Broadcast: 0 + 256 - 1 = 10.0.0.255
```

**Exemplo NetPractice (Level 2):**
```
C1: 127.0.0.1 / 255.255.255.252 (/30)
D1: 127.0.0.4 / 255.255.255.252 (/30)

Análise:
- Bloco = 256 - 252 = 4
- C1 rede: 127.0.0.0/30, broadcast: 127.0.0.3
  Hosts válidos: 127.0.0.1, 127.0.0.2
- D1 = 127.0.0.4?
  127.0.0.4 / 4 = 1 → rede começa em 1 × 4 = 4
  Rede D1: 127.0.0.4/30
  → D1 é NETWORK ADDRESS (inválido para host)!
  
Correção: D1 deve ser 127.0.0.2 (está em 127.0.0.0/30 como C1)
```

---

### 6️⃣ Cálculo: Determinação de Hosts Válidos em uma Sub-rede

**Por que fazer:**
- Você precisa saber qual é o intervalo de IPs que pode atribuir a máquinas.
- Hosts válidos = todos exceto network e broadcast.

**Como fazer:**
```
Hosts válidos: de (Network + 1) até (Broadcast - 1)
```

**Exemplo:** Sub-rede `192.168.1.128/26`
```
Network: 192.168.1.128
Broadcast: 192.168.1.191

Hosts válidos: 192.168.1.129 até 192.168.1.190
Total: 62 hosts possíveis
```

**Fórmula de quantidade de hosts:**
```
Número de hosts = Bloco - 2
(O -2 é para remover network e broadcast)
```

**Exemplo:** `/26` (bloco 64)
```
Hosts: 64 - 2 = 62
```

---

### 7️⃣ Cálculo: Validação de IP (É válido? É rede? É broadcast?)

**Por que fazer:**
- Um IP pode parecer válido (octetos 0–255) mas ser na verdade network ou broadcast.
- Você precisa validar antes de preencher a interface.

**Como fazer:**

#### Passo 1: Validar octetos
```
Cada octeto entre 0–255?
Se não → IP inválido
```

#### Passo 2: Calcular network da máscara
```
Rede = IP AND Máscara (usando bloco)
```

#### Passo 3: Verificar se é network
```
IP é network se: IP == Network
Exemplo: 192.168.1.128 com /26 (rede 192.168.1.128)
→ É network ❌
```

#### Passo 4: Verificar se é broadcast
```
IP é broadcast se: IP == Broadcast
Exemplo: 192.168.1.191 com /26 (broadcast 192.168.1.191)
→ É broadcast ❌
```

#### Passo 5: Se passou nos testes, é válido
```
IP é host válido ✅
```

**Exemplo completo:** IP `192.168.1.130` com `/26`
```
Octetos: 192, 168, 1, 130 → todos 0–255 ✓
Rede: 192.168.1.128
Broadcast: 192.168.1.191
130 ≠ 128 (não é network) ✓
130 ≠ 191 (não é broadcast) ✓
→ IP válido ✅
```

**Exemplo NetPractice (Level 1):**
```
A1: 104.98.23.13 / 255.255.255.0 (/24)
B1: 104.98.23.12 / 255.255.255.0 (/24)

Cálculo:
- Bloco = 256 - 0 = 256
- A1 rede: 104.98.23.0, broadcast: 104.98.23.255
  Hosts válidos: 104.98.23.1 até 104.98.23.254
  13 está em range ✓
- B1 rede: 104.98.23.0, broadcast: 104.98.23.255
  Hosts válidos: 104.98.23.1 até 104.98.23.254
  12 está em range ✓
  
Ambos têm 254 hosts possíveis: 104.98.23.1 até 104.98.23.254
```

---

### 7️⃣ Cálculo: Validação de IP (É válido? É rede? É broadcast?)

**Por que fazer:**
- Um IP pode parecer válido (octetos 0–255) mas ser na verdade network ou broadcast.
- Você precisa validar antes de preencher a interface.

**Como fazer:**

#### Passo 1: Validar octetos
```
Cada octeto entre 0–255?
Se não → IP inválido
```

#### Passo 2: Calcular network da máscara
```
Rede = IP AND Máscara (usando bloco)
```

#### Passo 3: Verificar se é network
```
IP é network se: IP == Network
Exemplo: 192.168.1.128 com /26 (rede 192.168.1.128)
→ É network ❌
```

#### Passo 4: Verificar se é broadcast
```
IP é broadcast se: IP == Broadcast
Exemplo: 192.168.1.191 com /26 (broadcast 192.168.1.191)
→ É broadcast ❌
```

#### Passo 5: Se passou nos testes, é válido
```
IP é host válido ✅
```

**Exemplo completo:** IP `192.168.1.130` com `/26`
```
Octetos: 192, 168, 1, 130 → todos 0–255 ✓
Rede: 192.168.1.128
Broadcast: 192.168.1.191
130 ≠ 128 (não é network) ✓
130 ≠ 191 (não é broadcast) ✓
→ IP válido ✅
```

**Exemplo NetPractice (Level 1):**
```
A1: 104.93.23.364 / 255.255.255.0
Passo 1: octeto 364 > 255 ❌ → INVÁLIDO

B1: 104.98.23.12 / 255.255.255.0
Passo 1: octetos 104, 98, 23, 12 → todos 0–255 ✓
Passo 2: rede = 104.98.23.0 (bloco 256)
Passo 3: 12 ≠ 0 (não é network) ✓
Passo 4: 12 ≠ 255 (não é broadcast) ✓
→ IP válido ✅
```

---

### 8️⃣ Cálculo: Verificar se dois IPs estão na MESMA rede

**Por que fazer:**
- Se dois IPs estão na mesma rede, comunicam direto (sem gateway).
- Se estão em redes diferentes, precisam de roteador.

**Como fazer:**

```
Passo 1: Calcule rede de IP1
Passo 2: Calcule rede de IP2
Passo 3: Compare
  Se rede1 == rede2 → MESMA REDE (direto)
  Se rede1 ≠ rede2 → REDES DIFERENTES (precisa gateway/rota)
```

**Importante:** a máscara deve ser a mesma nos dois IPs!

**Exemplo:** 
```
IP1: 192.168.1.10 / 255.255.255.0 → rede 192.168.1.0/24
IP2: 192.168.1.50 / 255.255.255.0 → rede 192.168.1.0/24
Resultado: MESMA REDE ✅ (podem falar direto)

IP1: 192.168.1.10 / 255.255.255.0 → rede 192.168.1.0/24
IP2: 192.168.2.10 / 255.255.255.0 → rede 192.168.2.0/24
Resultado: REDES DIFERENTES ❌ (precisam gateway)
```

**Exemplo NetPractice (Level 1):**
```
A1: 104.98.23.13 / 255.255.255.0 → rede 104.98.23.0/24
B1: 104.98.23.12 / 255.255.255.0 → rede 104.98.23.0/24
Resultado: MESMA REDE ✅ (conseguem falar direto, sem roteador)

C1: 211.191.29.75 / 255.255.0.0 → rede 211.191.0.0/16
D1: 211.190.301.42 / 255.255.0.0 ← inválido (octeto 301)
Correção: D1 deve ser 211.191.30.42 → rede 211.191.0.0/16
Resultado: MESMA REDE ✅ (conseguem falar direto)
```

---

### 9️⃣ Cálculo: Quantidade de Sub-redes em um /XX

**Por que fazer:**
- Entender quantas redes diferentes você consegue criar dentro de um intervalo.
- Útil para validar coerência do cenário.

**Como fazer:**
```
Número de sub-redes = 2 ^ (bits_de_host_acima_do_octeto)
```

**Exemplo:** `/24` em um `/16`
```
/16 tem 16 bits de rede, /24 tem 24 bits de rede
Diferença: 24 - 16 = 8 bits para subnetting
Sub-redes: 2^8 = 256 redes possíveis dentro de um /16
```

**Exemplo NetPractice (níveis avançados):**
```
Cenário: Roteador conecta múltiplas redes /26 dentro de /24
/24 = 256 endereços
/26 = 4 endereços por rede
Sub-redes possíveis: 256 / 4 = 4 redes /26 dentro de /24
  192.168.1.0/26 (0-63)
  192.168.1.64/26 (64-127)
  192.168.1.128/26 (128-191)
  192.168.1.192/26 (192-255)
```

---

### 🔟 Cálculo: Validação de Gateway

**Por que fazer:**
- Gateway (próximo salto) precisa estar na MESMA sub-rede do host.
- Caso contrário, o host não consegue alcançar o gateway.

**Como fazer:**

```
Passo 1: Calcule rede do host
Passo 2: Calcule rede do gateway (com mesma máscara)
Passo 3: Verifique se são iguais
  Se rede_host == rede_gateway → OK, gateway é acessível
  Se rede_host ≠ rede_gateway → ERRO, gateway não é alcançável
```

**Exemplo correto:**
```
Host: 192.168.1.50 / 255.255.255.0 → rede 192.168.1.0/24
Gateway: 192.168.1.1 / 255.255.255.0 → rede 192.168.1.0/24
Resultado: OK ✅ (host consegue alcançar gateway)
```

**Exemplo errado:**
```
Host: 192.168.1.50 / 255.255.255.0 → rede 192.168.1.0/24
Gateway: 192.168.2.1 / 255.255.255.0 → rede 192.168.2.0/24
Resultado: ERRO ❌ (host não consegue alcançar gateway)
```

**Exemplo NetPractice (Nível com roteador):**
```
Host A: 10.0.0.5 / 255.255.255.0 (rede 10.0.0.0/24)
Gateway padrão de A: 10.0.0.1

Validação:
- Rede de A: 10.0.0.0/24
- Rede de gateway: 10.0.0.0/24 (com mesma máscara)
- Resultado: OK ✅ (gateway é alcançável)
```

---

### 1️⃣1️⃣ Cálculo: Determinação de Rota (Reachability)

**Por que fazer:**
- Mesmo com gateway correto, o roteador precisa SABER como chegar na rede de destino.
- Sem rota explícita, ocorre erro "No forward way".

**Como fazer:**

```
Passo 1: Identifique rede de origem e rede de destino
Passo 2: Veja o roteador tem rota para destino
  Rota existente: "Se destino em X, encaminhe via interface Y"
Passo 3: Verifique se a interface Y consegue alcançar destino
  Interface do roteador deve estar na mesma rede do destino
Resultado: Se tudo OK, fluxo passa
```

**Exemplo (cenário 3 redes):**
```
Host A: 10.0.0.5/24 quer falar com Host C: 172.16.0.5/24
Roteador (R1):
  - Interface 1 (eth0): 10.0.0.1/24 (conecta rede de A)
  - Interface 2 (eth1): 172.16.0.1/24 (conecta rede de C)

Rota em R1: "Se destino em 172.16.0.0/24, encaminhe via eth1"

Fluxo:
1. A envia para gateway 10.0.0.1
2. R1 recebe em eth0
3. R1 procura rota para 172.16.0.5 → encontra "via eth1"
4. R1 encaminha pela eth1 (172.16.0.1)
5. C recebe ✅
```

**Exemplo NetPractice (Nível com 2+ roteadores):**
```
Host A: 10.0.0.5 quer falar com Host D: 192.168.1.5

Caminho esperado:
- A (10.0.0.0/24) → gateway R1 (10.0.0.1)
- R1 recebe em eth0, procura rota para 192.168.1.0/24
- R1 encontra rota: eth1 conecta 192.168.1.0/24
- R1 encaminha via eth1 (192.168.1.1)
- D recebe em sua interface

Validação:
- R1 tem interface em 10.0.0.0/24? ✓
- R1 tem interface em 192.168.1.0/24? ✓
- R1 tem rota "dest 192.168.1.0/24 via eth1"? ✓
- D conhece gateway para responder? ✓
```

---

### 1️⃣2️⃣ Cálculo: Verificar Octeto Quebrado (Qual é?)

**Por que fazer:**
- O octeto quebrado determina qual será o "salto" entre sub-redes.
- Se você não o identifica corretamente, todo o resto do cálculo erra.

**Como fazer:**

```
Verifique cada octeto da máscara:
- Se é 255 → completamente de rede, passe pro próximo
- Se é 0 → completamente de host, passe pro próximo
- Se é outro (192, 224, 240, etc.) → esse é o quebrado, PARE
```

**Exemplos:**
```
Máscara 255.255.255.0 → octeto quebrado: nenhum! (é /24, bloco 256)
Máscara 255.255.255.128 → octeto quebrado: o último (128)
Máscara 255.255.0.0 → octeto quebrado: nenhum! (é /16, bloco 256 no terceiro octeto)
Máscara 255.255.255.240 → octeto quebrado: o último (240)
```

**Exemplo NetPractice (Level 3+):**
```
Interface: 192.168.1.70 / 255.255.255.192
Octeto quebrado?
- 1º octeto: 255 → de rede
- 2º octeto: 255 → de rede
- 3º octeto: 255 → de rede
- 4º octeto: 192 → não é 255 nem 0 → É O QUEBRADO!

Bloco: 256 - 192 = 64
Sub-redes no último octeto: 0, 64, 128, 192
70 cai em: 70 / 64 = 1 → 1 × 64 = 64
Rede: 192.168.1.64/26
```

---

### 1️⃣3️⃣ Cálculo: AND Binário (Teórico Puro)

**Por que fazer:**
- É o fundamento matemático de como calcular rede.
- Não é obrigatório saber em detalhe, mas entender ajuda.

**Como fazer:**

```
AND bit por bit:
1 AND 1 = 1
1 AND 0 = 0
0 AND 1 = 0
0 AND 0 = 0

IP AND Máscara = Rede
```

**Exemplo:** IP `192.168.1.130`, Máscara `255.255.255.192`
```
Último octeto em binário:
IP:      10000010 (130)
Máscara: 11000000 (192)
AND:     10000000 (128)

Resultado: 192.168.1.128
```

**Por que o método do bloco é mais rápido:** você não precisa converter pra binário; é apenas divisão e multiplicação.

---

### Resumo: Qual Cálculo Usar Quando?

| Situação | Cálculo |
|----------|---------|
| "Que prefixo é 255.255.255.192?" | #2 (decimal → CIDR) |
| "Que máscara é /26?" | #1 (CIDR → decimal) |
| "Qual é a rede de 10.20.50.130/25?" | #4 (bloco) |
| "Qual é o broadcast de 10.20.50.130/25?" | #5 (network + bloco - 1) |
| "Quais IPs posso usar como host em 10.20.50.0/26?" | #6 (network+1 até broadcast-1) |
| "192.168.1.0 com /24 é válido para host?" | #7 (validação) |
| "192.168.1.50 e 192.168.2.50 falam direto?" | #8 (mesma rede?) |
| "192.168.1.254 é gateway válido para 192.168.1.50?" | #10 (mesmo octeto de rede?) |
| "Como host A alcança host C via R1?" | #11 (reachability) |

---

## Roteamento e Gateways

### O que é um Gateway?
Um **gateway** (ou **roteador**) é um dispositivo que conecta redes diferentes e encaminha pacotes entre elas.

### Tabela de Roteamento
Cada dispositivo tem uma **tabela de roteamento** que diz:
- "Se o destino está em [rede X], envie para [gateway Y]"

**Exemplo:**
```
Destino: 192.168.1.0/24  → Gateway: 192.168.1.1
Destino: 192.168.2.0/24  → Gateway: 192.168.1.1
Destino: 10.0.0.0/8      → Gateway: 192.168.1.254
Destino: 0.0.0.0/0       → Gateway: 192.168.1.1 (rota padrão, internet)
```

### Configuração Mínima para 2 Redes Falarem

Se host A (`10.0.0.5/24`) quer falar com host B (`192.168.1.10/24`):

1. **Host A:**
   - IP: `10.0.0.5/24`
   - Gateway: `10.0.0.1` (roteador que sabe como chegar em `192.168.1.0/24`)

2. **Host B:**
   - IP: `192.168.1.10/24`
   - Gateway: `192.168.1.1` (roteador que sabe como chegar em `10.0.0.0/24`)

3. **Roteador (interface A):**
   - IP: `10.0.0.1/24` (mesma rede que host A)
   - Interface B: `192.168.1.1/24` (mesma rede que host B)

4. **Rota no roteador:**
   - De `10.0.0.0/24` para `192.168.1.0/24` (sabe como alcançar)

**Resultado:** 
- Host A → gateway `10.0.0.1` → roteador → gateway `192.168.1.1` → Host B

### Erro Comum: "No forward way"
Significa:
- Host A quer mandar pacote para host B
- Mas host B está em rede diferente
- E não há rota configurada
- Ou o gateway não consegue alcançar a rede de B

---

## Método de Resolução (Checklist)

Aplique esse **checklist de 5 passos** para cada link/interface do NetPractice:

### 📋 Checklist Ultra Rápido (30 seg por link)

```
1. ✅ IP válido?
   → Cada octeto entre 0-255?
   → Se não → ERRO, ajustar IP

2. ✅ Máscara válida?
   → É um valor reconhecido (/24, /25, /16, etc.)?
   → Se máscara em decimal, soma de bits 1 = CIDR?

3. ✅ Calcular rede e broadcast
   → Qual é a rede deste IP?
   → Qual é o broadcast?
   → Hosts válidos: rede+1 até broadcast-1

4. ✅ Comparar redes
   → Este host fala com o outro em uma interface?
   → Se SIM e redes diferentes → ERRO, deve ter gateway
   → Se SIM e redes iguais → OK, podem falar direto
   → Se NÃO (redes diferentes) → OK se há roteador configurado

5. ✅ Sem duplicatas
   → Nenhum host usa o mesmo IP?
   → Nenhum host usa rede ou broadcast como IP?
```

### 🔍 Detalhado por Tipo de Problema

#### Tipo A: Dois hosts na mesma rede devem falar direto

```
Checklist:
1. IP A válido? IP B válido?
2. Máscara A e B iguais?
3. Calcular rede de A e rede de B
4. Redes A == B?
   → SIM: OK, falam direto
   → NÃO: ERRO, devem estar na mesma rede ou usar gateway
5. A é rede/broadcast? B é rede/broadcast?
   → SIM para qualquer: ERRO, mudar para host válido
```

#### Tipo B: Dois hosts em redes diferentes comunicam via roteador

```
Checklist:
1. IPs dos hosts válidos? IPs dos gateways válidos?
2. Host A tem gateway apontando para roteador? (interface do roteador em rede de A)
3. Host B tem gateway apontando para roteador? (interface do roteador em rede de B)
4. Roteador tem duas interfaces em redes diferentes?
5. Roteador tem rota para chegar em rede de B a partir de interface que conecta com A?
```

#### Tipo C: Roteador conectando múltiplas redes

```
Checklist:
1. Cada interface do roteador está em uma rede diferente?
2. Cada interface tem IP e máscara válidos?
3. Nenhuma interface usa rede/broadcast?
4. Tabela de roteamento: sabe chegar em todas as redes?
5. Cada host conhece seu gateway?
```

---

## Frases para Avaliação

Use essas frases/estrutura para explicar sua resolução:

### 📢 Explicação de Uma Interface

```
"Este IP é 192.168.1.130 com máscara /26 (255.255.255.192).

A máscara /26 significa 26 bits de rede e 6 bits de host.

Calculando a rede:
- Bloco = 256 - 192 = 64
- 130 / 64 = 2, então rede começa em 2 × 64 = 128
- Rede: 192.168.1.128
- Broadcast: 192.168.1.191 (128 + 64 - 1)
- Hosts válidos: 192.168.1.129 até 192.168.1.190

Portanto, este IP é válido (está entre 129 e 190) e a interface faz parte da rede 192.168.1.128/26."
```

### 📢 Explicação de Conectividade Entre Dois Hosts

```
"Host A tem IP 192.168.1.10/24 e Host B tem IP 192.168.2.10/24.

Para Host A:
- Rede: 192.168.1.0/24
- Broadcast: 192.168.1.255

Para Host B:
- Rede: 192.168.2.0/24
- Broadcast: 192.168.2.255

Como as redes são diferentes (192.168.1.0 ≠ 192.168.2.0), 
os hosts NÃO podem falar direto. 

Para que se comuniquem, é necessário um roteador (gateway) 
com interfaces em ambas as redes e uma rota de encaminhamento."
```

### 📢 Explicação de Roteador

```
"O roteador conecta três redes diferentes:

Interface 1: 10.0.0.1/24 (rede 10.0.0.0/24) — conecta com host A
Interface 2: 192.168.1.1/24 (rede 192.168.1.0/24) — conecta com host B
Interface 3: 172.16.0.1/16 (rede 172.16.0.0/16) — conecta com host C

Cada interface está em uma rede diferente.

Quando Host A (10.0.0.5) quer falar com Host B (192.168.1.10):
1. Host A envia pacote para seu gateway: 10.0.0.1 (roteador)
2. Roteador recebe na interface 1
3. Roteador sabe que destino 192.168.1.10 é acessível via interface 2
4. Roteador encaminha para interface 2 (192.168.1.1)
5. Host B recebe o pacote (está na mesma rede 192.168.1.0/24)

Portanto, a configuração está correta."
```

### 📢 Por que CIDR/Máscara Importa

```
"A máscara define a 'fronteira' entre rede e host.

Se eu tiver máscara /24 (255.255.255.0), o último octeto é inteiro de host,
então uma rede pode ter até 254 hosts (de .1 até .254).

Se eu tiver máscara /25 (255.255.255.128), o último bit do último octeto é de rede,
então divido cada /24 em duas sub-redes de 126 hosts cada.

Isso é importante porque define:
- Quantos hosts cabem em uma rede
- Qual IP pertence a qual rede
- Se dois IPs podem falar direto"
```

---

## Exemplos Resolvidos

### Exemplo 1: Level 1 (Básico)

**Enunciado:**
- Host A (my PC) precisa comunicar com Host B (little brother's computer)
- Host C (my Mac) precisa comunicar com Host D (little sister's computer)
- Status: KO — "No forward way"

**Dados iniciais:**
```
Interface A1: IP 104.93.23.364 / Mask 255.255.255.0 ❌ (364 > 255, inválido)
Interface B1: IP 104.98.23.12 / Mask 255.255.255.0 ✓
Interface C1: IP 211.191.29.75 / Mask 255.255.0.0 ✓
Interface D1: IP 211.190.301.42 / Mask 255.255.0.0 ❌ (301 > 255, inválido)
```

**Análise:**
1. **A1 inválido:** octeto `.364` não existe (máximo 255)
2. **D1 inválido:** octeto `.301` não existe

**Passo 1: Corrigir A1**
```
A1 deve estar na mesma rede que B1 (máscara /24 = 255.255.255.0)
B1 rede: 104.98.23.0 (primeiros 3 octetos)

Então A1 deve começar com 104.98.23.X
Escolho: 104.98.23.13 (qualquer entre .1 e .254, menos .0 e .255)
```

**Passo 2: Corrigir D1**
```
D1 deve estar na mesma rede que C1 (máscara /16 = 255.255.0.0)
C1 rede: 211.191.0.0 (primeiros 2 octetos)

Então D1 deve começar com 211.191.X.X
Escolho: 211.191.30.42 (mantendo o último octeto, corrigindo o terceiro)
```

**Solução Final:**
```
Interface A1: 104.98.23.13 / 255.255.255.0  ✅
Interface B1: 104.98.23.12 / 255.255.255.0  ✅
Interface C1: 211.191.29.75 / 255.255.0.0   ✅
Interface D1: 211.191.30.42 / 255.255.0.0   ✅
```

**Prova:**
- A1 rede = 104.98.23.0, B1 rede = 104.98.23.0 → **mesma rede ✅**
- C1 rede = 211.191.0.0, D1 rede = 211.191.0.0 → **mesma rede ✅**

### Guia Operacional (este Level 1 do print) — passo a passo detalhado

Use este roteiro exatamente na interface do NetPractice:

1. **Leia os objetivos no topo**
   - Objetivo 1: host A falar com host B
   - Objetivo 2: host C falar com host D
   - Não há roteador no cenário, então cada par precisa estar na **mesma sub-rede**.

2. **Valide os 4 IPs antes de qualquer cálculo**
   - A1 = `104.93.23.364` → inválido (`364 > 255`)
   - B1 = `104.98.23.12` → válido
   - C1 = `211.191.29.75` → válido
   - D1 = `211.190.301.42` → inválido (`301 > 255`)

3. **Resolva o par A-B (máscara /24)**
   - B1 está em `/24` (`255.255.255.0`), então a rede é `104.98.23.0/24`.
   - Para A falar com B sem roteador, A1 também deve ser `104.98.23.X`.
   - Escolha um host válido (não `.0`, não `.255`, não repetido):  
     **A1 = `104.98.23.13`**.

4. **Resolva o par C-D (máscara /16)**
   - C1 está em `/16` (`255.255.0.0`), então a rede é `211.191.0.0/16`.
   - Para C falar com D sem roteador, D1 também deve começar com `211.191`.
   - Ajuste D1 para um host válido:  
     **D1 = `211.191.30.42`**.

5. **Preenchimento final no level**
   - A1: `104.98.23.13 / 255.255.255.0`
   - B1: `104.98.23.12 / 255.255.255.0` (mantém)
   - C1: `211.191.29.75 / 255.255.0.0` (mantém)
   - D1: `211.191.30.42 / 255.255.0.0`

6. **Como justificar na avaliação (explicação curta e forte)**
   - \"Primeiro removi IPs inválidos (>255).  
   Depois forcei cada par para a mesma rede, pois não existe roteador no cenário.  
   A-B em `104.98.23.0/24` e C-D em `211.191.0.0/16`.  
   Também garanti que os IPs escolhidos são hosts válidos (não rede, não broadcast, não duplicados).\"  

7. **Erros clássicos desse level**
   - Corrigir só o octeto inválido e esquecer a sub-rede.
   - Manter A1 como `104.93...` enquanto B1 está em `104.98...` com `/24`.
   - Achar que por ter \"104\" no início já é a mesma rede (não é; com `/24`, importam os 3 primeiros octetos).

---

### Exemplo 2: Cenário com /25

**Cenário:**
- Host X: `192.168.1.10` com máscara `255.255.255.128` (/25)
- Host Y: `192.168.1.200` com máscara `255.255.255.128` (/25)
- Pergunta: Conseguem falar direto?

**Cálculo Host X:**
```
IP: 192.168.1.10
Máscara: /25 (255.255.255.128)
Octeto quebrado: último (128)
Bloco: 256 - 128 = 128
Posição: 10 / 128 = 0.078... → 0
Rede: 0 × 128 = 0
Rede X: 192.168.1.0/25
Broadcast: 192.168.1.127
Hosts válidos: 192.168.1.1 até 192.168.1.126
Host X (10) é válido ✅
```

**Cálculo Host Y:**
```
IP: 192.168.1.200
Máscara: /25 (255.255.255.128)
Octeto quebrado: último (128)
Bloco: 256 - 128 = 128
Posição: 200 / 128 = 1.56... → 1
Rede: 1 × 128 = 128
Rede Y: 192.168.1.128/25
Broadcast: 192.168.1.255
Hosts válidos: 192.168.1.129 até 192.168.1.254
Host Y (200) é válido ✅
```

**Resultado:**
```
Rede X = 192.168.1.0/25
Rede Y = 192.168.1.128/25

Redes diferentes! 
→ Não conseguem falar direto
→ Precisam de roteador/gateway
```

---

### Exemplo 3: Cenário com /26 e Roteador

**Rede A:**
- Host A1: `10.0.0.5/26`
- Host A2: `10.0.0.50/26`

**Rede B:**
- Host B1: `10.0.0.130/26`
- Host B2: `10.0.0.200/26`

**Roteador (intermediário):**
- Interface R1: `10.0.0.1/26`
- Interface R2: `10.0.0.129/26`
- Tabela de rota: sabe alcançar ambas redes

**Análise:**

Rede A (máscara /26, bloco 64):
```
IPs: 10.0.0.5 e 10.0.0.50
Octeto: 5 / 64 = 0, rede = 0
Octeto: 50 / 64 = 0, rede = 0
Rede A: 10.0.0.0/26
Broadcast: 10.0.0.63
A1 (5) ✓ e A2 (50) ✓ estão em 10.0.0.0/26
```

Rede B:
```
IPs: 10.0.0.130 e 10.0.0.200
Octeto: 130 / 64 = 2, rede = 128
Octeto: 200 / 64 = 3, rede = 192
PROBLEMA! 130 está em 10.0.0.128/26, mas 200 está em 10.0.0.192/26
```

**Correção:**
```
Mudar B2 para 10.0.0.150 (também em 10.0.0.128/26):
Octeto: 150 / 64 = 2, rede = 128 ✓

Ou mudar para /25 para caber todos na mesma rede
```

**Assumindo B2 = `10.0.0.150/26`:**

```
Roteador:
- Interface R1: 10.0.0.1/26 (rede A: 10.0.0.0/26) ✓
- Interface R2: 10.0.0.129/26 (rede B: 10.0.0.128/26) ✓

Host A1 (10.0.0.5) quer falar com B1 (10.0.0.130):
1. A1 vê que B1 não está em 10.0.0.0/26
2. A1 envia para gateway: 10.0.0.1 (roteador)
3. Roteador sabe que 10.0.0.128/26 é acessível via interface R2
4. Encaminha para B1 via interface R2
5. B1 recebe ✓

Conectividade: OK ✓
```

---

## Dicas Extras

### Truque: Memorizar rápido as máscaras
```
/24 = .0     (255.255.255.0)
/25 = .128   (256/2 = 128)
/26 = .192   (256/2 + 256/4 = 128 + 64)
/27 = .224   (128 + 64 + 32)
/28 = .240   (128 + 64 + 32 + 16)
/29 = .248   (128 + 64 + 32 + 16 + 8)
/30 = .252   (128 + 64 + 32 + 16 + 8 + 4)
```

### Truque: Testar rápido se um IP é network/broadcast
```
IP é network?
→ Todos os bits de host = 0
→ Último octeto é múltiplo perfeito do bloco?

IP é broadcast?
→ Todos os bits de host = 1
→ Último octeto = network + (bloco - 1)?
```

### Truque: Ordem de análise em nível complexo
Quando o cenário tem vários hosts e roteadores, siga sempre a mesma ordem.  
Isso evita erro de lógica (ex.: tentar corrigir rota antes de perceber IP inválido).

#### Passo 1) Validar TODOS os IPs (0-255 por octeto)
**Como fazer:**
- Verifique interface por interface.
- Cada IP deve ter 4 octetos e cada octeto deve estar em `0..255`.

**Teoria por trás:**
- IPv4 tem 32 bits (4 octetos de 8 bits).  
- Um octeto de 8 bits só representa valores de 0 a 255.  
- Se aparecer `301`, `364`, `-1`, etc., o endereço nem existe na pilha IP.

**Erro típico que esse passo evita:** perder tempo com máscara/rota de um IP inválido.

#### Passo 2) Validar TODAS as máscaras
**Como fazer:**
- Confirme se a máscara é válida em CIDR contínuo (bits 1 seguidos de bits 0).
- Máscaras comuns no NetPractice: `/8`, `/16`, `/24`, `/25`, `/26`, `/27`, `/28`, `/29`, `/30`.
- Em decimal, valores válidos por octeto são: `255, 254, 252, 248, 240, 224, 192, 128, 0`.

**Teoria por trás:**
- A máscara define fronteira entre **rede** e **host**.
- Sem máscara válida, você não consegue determinar rede, broadcast e roteamento corretamente.

**Erro típico que esse passo evita:** assumir que dois hosts estão na mesma rede só pelo “formato” do IP.

#### Passo 3) Calcular TODAS as redes
**Como fazer:**
- Para cada interface, calcule:
  - endereço de rede
  - broadcast
  - faixa de hosts válidos
- Método rápido: no octeto quebrado, `bloco = 256 - máscara`.

**Teoria por trás:**
- A decisão “fala direto ou via roteador” depende da rede resultante (`IP AND máscara`).
- Endereço de rede e broadcast são reservados e não podem ser atribuídos a hosts.

**Mini-exemplo:**
- `192.168.10.70/26` -> máscara `255.255.255.192`, bloco `64`
- Redes possíveis no último octeto: `0, 64, 128, 192`
- 70 cai no bloco `64-127`:
  - rede = `192.168.10.64`
  - broadcast = `192.168.10.127`
  - hosts válidos = `.65` a `.126`

#### Passo 4) Listar TODAS as conectividades requeridas
**Como fazer:**
- Leia o enunciado e anote os objetivos de comunicação (ex.: A -> B, C -> D).
- Trate cada objetivo como um “teste” independente.

**Teoria por trás:**
- Em redes, “estar configurado” não basta; você precisa validar o **fluxo de origem -> destino**.
- Um cenário pode ter parte funcionando e parte quebrada.

**Erro típico que esse passo evita:** corrigir só um objetivo e esquecer os outros.

#### Passo 5) Identificar quem fala com quem (caminho)
**Como fazer:**
- Para cada objetivo, desenhe o caminho lógico:
  - origem
  - interface de saída da origem
  - gateway (se houver)
  - roteador(es)
  - rede de destino
  - host destino

**Teoria por trás:**
- Pacote IP sempre segue decisão de roteamento por salto.
- Se você não conhece o caminho, não sabe onde o tráfego está quebrando.

**Erro típico que esse passo evita:** culpar “a rede inteira” quando o problema é apenas um gateway.

#### Passo 6) Checar se conseguem (mesma rede) ou se faltam gateways
**Como fazer:**
- Se origem e destino estão na mesma rede: comunicação direta (sem gateway).
- Se estão em redes diferentes: origem precisa de gateway válido na própria rede.
- O roteador também precisa de interface/rota para continuar o encaminhamento.

**Teoria por trás:**
- Host só envia direto para quem está na mesma sub-rede (mesmo domínio L2).
- Para outra rede, o próximo salto é o gateway padrão.

**Checklist rápido deste passo:**
- gateway pertence à mesma rede do host?
- interface do roteador no lado de destino está correta?
- existe rota até a rede de destino?

#### Passo 7) Descrever tabelas de rota (se necessário)
**Como fazer:**
- Quando houver roteadores, explicite rotas no formato:
  - `destino/prefixo -> next hop` (ou interface de saída).
- Inclua rota padrão (`0.0.0.0/0`) quando for o caso.

**Teoria por trás:**
- Roteador não “adivinha” caminho: ele consulta tabela de rotas.
- Sem rota correspondente, ocorre erro de encaminhamento (`No forward way`).

**Exemplo conceitual:**
- R1 conectado em `10.0.0.0/24` e `192.168.1.0/24`
- Rota em host de `10.0.0.5`: default -> `10.0.0.1`
- Rota em host de `192.168.1.8`: default -> `192.168.1.1`
- Resultado: tráfego cruza redes via R1.

#### Regra de ouro para prova
Sempre corrija nesta ordem:
1. IP inválido  
2. máscara inválida  
3. rede/broadcast/duplicado  
4. gateway  
5. rota  

Essa sequência reduz muito retrabalho e te dá uma explicação sólida na avaliação.

---

### Guia Operacional — Level 2 (Router conectando 2 redes)

**Enunciado:**
- Goal 1: host B (Computer B) needs to communicate with host A (Computer A)
- Goal 2: host D (Computer D) needs to communicate with host C (Computer C)
- Status: KO — "No forward way"

**Dados iniciais do print:**
```
Interface A1: IP 192.168.29.1 / Mask 255.255.255.224
Interface B1: IP 192.168.29.222 / Mask 255.255.255.32
Interface C1: IP 127.0.0.1 / Mask 255.255.255.252
Interface D1: IP 127.0.0.4 / Mask /30
```

#### Aplicando os 7 Passos:

**Passo 1) Validar TODOS os IPs**
```
A1: 192.168.29.1 ✓ válido
B1: 192.168.29.222 ✓ válido
C1: 127.0.0.1 ✓ válido
D1: 127.0.0.4 ✓ válido
```

**Passo 2) Validar TODAS as máscaras**
```
A1: 255.255.255.224 ✓ válida (/27)
B1: 255.255.255.32 ❌ INVÁLIDA! (32 não é um valor válido de octeto de máscara)
     Valores válidos: 255, 254, 252, 248, 240, 224, 192, 128, 0
C1: 255.255.255.252 ✓ válida (/30)
D1: /30 (255.255.255.252) ✓ válida
```

**Problema encontrado:** B1 tem máscara inválida.

**Passo 3) Calcular TODAS as redes**

Para A1 (192.168.29.1/27):
```
Máscara: 255.255.255.224 → bloco = 256 - 224 = 32
IP octeto: 1
Posição: 1 / 32 = 0 → 0
Rede A1: 192.168.29.0/27
Broadcast: 192.168.29.31
Hosts válidos: 192.168.29.1 até 192.168.29.30
A1 (1) é válido ✓
```

Para B1 com máscara corrigida para /27 (192.168.29.222/27):
```
Máscara: 255.255.255.224 → bloco = 32
IP octeto: 222
Posição: 222 / 32 = 6 (arredonda) → 6
Rede: 6 × 32 = 192
Rede B1: 192.168.29.192/27
Broadcast: 192.168.29.223
Hosts válidos: 192.168.29.193 até 192.168.29.222
B1 (222) é válido ✓ MAS em rede diferente de A1!
```

Para C1 (127.0.0.1/30):
```
Máscara: 255.255.255.252 → bloco = 256 - 252 = 4
IP octeto: 1
Posição: 1 / 4 = 0 → 0
Rede C1: 127.0.0.0/30
Broadcast: 127.0.0.3
Hosts válidos: 127.0.0.1 até 127.0.0.2
C1 (1) é válido ✓
```

Para D1 (127.0.0.4/30):
```
Máscara: 255.255.255.252 → bloco = 4
IP octeto: 4
Posição: 4 / 4 = 1 → 1
Rede: 1 × 4 = 4
Rede D1: 127.0.0.4/30
Network address: 127.0.0.4
Broadcast: 127.0.0.7
D1 (4) é REDE, não host! ❌ Inválido
```

**Problemas encontrados:**
1. B1 tem máscara inválida (32 não é válido)
2. B1 (mesmo com máscara corrigida) está em rede diferente de A1
3. D1 é network address, não host válido

**Passo 4) Listar conectividades requeridas**
- Computer B falar com Computer A
- Computer D falar com Computer C

**Passo 5) Identificar caminho**
- B1 -> A1: são hosts diferentes, precisam estar na mesma rede ou haver roteador
- D1 -> C1: são hosts diferentes, precisam estar na mesma rede ou haver roteador

**Passo 6) Checar se conseguem**

Cenário atual (com correções de máscara):
- A1 está em `192.168.29.0/27`
- B1 estaria em `192.168.29.192/27` (rede diferente!)
- C1 está em `127.0.0.0/30`
- D1 seria invalid (é rede)

Conclusão: não conseguem comunicar direto; precisaríamos de roteador conectando ambas redes.  
Mas no diagrama não há roteador, então **ambos os pares precisam estar na MESMA rede**.

**Solução:**

Para o par A-B:
- A1: `192.168.29.1/27` (rede 192.168.29.0/27) — mantém
- B1: precisa estar em 192.168.29.0/27 também
  - Máscara corrigida: `255.255.255.224` (não 32)
  - IP: `192.168.29.2` (qualquer host válido em 1-30)
  
Para o par C-D:
- C1: `127.0.0.1/30` (rede 127.0.0.0/30) — mantém
- D1: precisa estar em 127.0.0.0/30 também
  - Máscara corrigida: `255.255.255.252` (já é /30)
  - IP: `127.0.0.2` (não pode ser 4, pois 4 é rede)

**Valores finais para preencher:**
```
A1: 192.168.29.1 / 255.255.255.224 (mantém)
B1: 192.168.29.2 / 255.255.255.224 (corrige IP e máscara)
C1: 127.0.0.1 / 255.255.255.252 (mantém)
D1: 127.0.0.2 / 255.255.255.252 (corrige IP e máscara)
```

**Prova:**
- A1 rede = 192.168.29.0/27, B1 rede = 192.168.29.0/27 → **mesma rede ✅**
- C1 rede = 127.0.0.0/30, D1 rede = 127.0.0.0/30 → **mesma rede ✅**

**Explicação para avaliação:**
"Primeiro identifiquei que B1 tinha máscara inválida (255.255.255.32 não existe).  
Depois calculei as redes: A1 e B1 eram versões diferentes de 192.168.29.0/27, então A estaria em 0-31 e B estaria em 192-223, redes diferentes.  
Como não há roteador, forcei ambos para mesma rede: A-B em 192.168.29.0/27 e C-D em 127.0.0.0/30.  
Também corrigi D1 que era network address (não host válido), movendo para 127.0.0.2."

**Erros clássicos desse level:**
- Esquecer que máscara .32 é inválida (confundir com IP válido).
- Tentar "consertar" B1 só o IP, mantendo máscara inválida.
- Não perceber que D1=4 é network address com /30.
- Usar 127.0.0.x sem saber que é loopback (mas neste level funciona como rede comum).

---

## Resumo Ultra-Rápido

| Conceito | Como resolver |
|----------|---|
| **IP inválido** | Octeto > 255? → Mudar para 0-255 |
| **Máscara inválida** | Não reconhece /XX? → Converter para decimal (tabela) |
| **Mesma rede?** | Calcular rede dos dois (IP AND máscara) → comparar |
| **Rede vs Broadcast** | Rede = bloco × piso(IP/bloco), Broadcast = rede+bloco-1 |
| **Conectividade direta** | Mesma rede + IPs válidos → OK |
| **Conectividade indireta** | Redes diferentes + gateway + rota → OK |
| **Gateway correto?** | Gateway deve estar na mesma rede do host |
| **Rota correta?** | Roteador sabe alcançar destino (tabela de rota) |

---

## Checklist Final Antes de Submeter

- [ ] Todo IP tem 4 octetos no intervalo 0-255?
- [ ] Toda máscara é válida (/8 até /32)?
- [ ] Nenhum IP é rede ou broadcast na sua rede?
- [ ] Dois hosts que devem falar estão na mesma rede ou têm gateway?
- [ ] Nenhum IP está duplicado?
- [ ] Se há roteador, cada interface está em rede diferente?
- [ ] Se há roteador, ele tem rota para alcançar todos os destinos?
- [ ] Cada host que precisa falar sabe seu gateway?

---

**Boa sorte! Você tem todo o conhecimento para resolver os 10 níveis. 💪**
