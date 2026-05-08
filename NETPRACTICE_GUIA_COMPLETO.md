# NetPractice — Guia Completo de Teoria e Resolução

> **Objetivo:** Dominar conceitos de redes TCP/IP e resolver todos os 10 níveis do NetPractice com confiança.

---

## ÍNDICE
1. [Conceitos Fundamentais](#conceitos-fundamentais)
2. [Endereçamento IPv4](#endereçamento-ipv4)
3. [Máscaras e CIDR](#máscaras-e-cidr)
4. [Cálculo de Redes](#cálculo-de-redes)
5. [Roteamento e Gateways](#roteamento-e-gateways)
6. [Método de Resolução (Checklist)](#método-de-resolução-checklist)
7. [Frases para Avaliação](#frases-para-avaliação)
8. [Exemplos Resolvidos](#exemplos-resolvidos)

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
```
1. Validar TODOS os IPs (0-255 por octeto)
2. Validar TODAS as máscaras
3. Calcular TODAS as redes
4. Listar TODAS as conectividades requeridas
5. Identificar quem fala com quem
6. Checar se conseguem (mesma rede) ou se faltam gateways
7. Descrever tabelas de rota se necessário
```

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
