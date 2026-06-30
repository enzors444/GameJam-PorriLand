# PorriLand — Balanceamento e Novas Mecânicas

**Data:** 2026-06-29
**Jam:** Overpower
**Plataforma:** Construct 3

---

## Filosofia Central

> *Fanatismo é uma faca de dois gumes: derrota na Era 1, risco calculado na Era 2, caminho válido (porém custoso) na Era 3.*

O jogador precisa ver onde está no espectro de Fé e ter ferramentas para escolher deliberadamente seu destino. As mudanças se organizam em três eixos:

1. **Legibilidade** — o jogador vê a Fé se aproximando de zonas de perigo
2. **Controle ativo** — novos verbos dão agência sobre a Fé
3. **Consequência narrativa** — o estado final + karma dos dilemas determina o final

---

## 1. Legibilidade da Fé

### Zonas visuais da barra de Fe

| Zona | Intervalo | Cor | Estado |
|------|-----------|-----|--------|
| Ceticismo | 0 – 30 | Azul frio | Ceticismo |
| Atenção baixa | 31 – 44 | Azul claro | Equilibrado (beira) |
| Equilibrado | 45 – 65 | Verde | Equilibrado |
| Atenção alta | 66 – 79 | Laranja | Equilibrado (beira) |
| Fanatismo | 80 – 100 | Vermelho | Fanatismo |

*Os limiares exatos de transição de `Estado` devem ser verificados no event sheet e os valores de zona ajustados para coincidir.*

### Avisos progressivos

- `Fe > 66`: ícone de chama pulsa no HUD
- `Fe > 79`: leve tint vermelho na tela + som de multidão agitada em loop suave
- Transição para Fanatismo: flash vermelho + texto breve "O POVO ESTÁ EXALTADO"
- Transição para Ceticismo: flash azul + texto "A FÉ ENFRAQUECE"

---

## 2. Novos Poderes

O sistema pipe-delimited existente (`Animacoes|IDs|Eras|Cooldowns`) é estendido com os novos poderes. Desbloqueios por era:

| Era | Poderes disponíveis |
|-----|-------------------|
| 1 | Raio + Bênção |
| 2 | + Sussurro + Redenção |
| 3 | + Silêncio + Chama |

### Poderes Essenciais

#### Bênção (Era 1, CD 8s)
- **Alvo:** Adepto específico (clique)
- **Efeito:** `Felicidade +25` (clampado a 100)
- **Função:** Estabelece o loop de cuidado desde o início; substitui a falta de agência direta sobre Adeptos individuais
- **Visual:** partícula dourada irradiando do Adepto

#### Silêncio (Era 3, CD 15s)
- **Alvo:** área vazia / céu (clique sem alvo)
- **Efeito:** `Fe −12` globalmente; todos os Adeptos ficam `Imune = 1` por 3s
- **Função:** escape valve do Fanatismo — deliberado, poderoso, disponível apenas no final
- **Visual:** onda branca se expande do centro da tela; todos os Porris pausam brevemente
- **Flavor:** o deus se cala, e o silêncio acalma os fiéis

### Poderes Opcionais (se houver tempo)

#### Redenção (Era 2, CD 10s)
- **Alvo:** Fanático específico (clique)
- **Efeito:** Fanático vira Adepto com `Felicidade = 50`; `Fe +5` (milagre custa Fé)
- **Função:** alternativa misericordiosa ao Raio; ativa flag `Raio_Nunca_Adepto` apenas se Raio nunca foi usado em Adepto
- **Visual:** luz branca descendo sobre o Fanático

#### Chama (Era 3, CD 20s)
- **Alvo:** Adepto específico (clique)
- **Efeito:** Adepto vira "Evangelista" por 15s — raio de conversão 2× e chance de conversão +10%
- **Função:** propaga fé positivamente; o verbo "inspirar"
- **Visual:** auréola laranja sobre o Adepto evangelista

---

## 3. Sistema de Karma

### Variável global
```
Karma = 0  (inicia em 0, não exibida ao jogador)
```

### Acúmulo por dilemas

| Dilema | Escolha | Delta Karma |
|--------|---------|-------------|
| Festa da Esperança | SIM (festa) | +2 |
| Festa da Esperança | NÃO (proíbe) | −2 |
| Justiça Divina | SIM (Raio no vizinho) | −3 |
| Justiça Divina | NÃO (perdoa) | +3 |
| Voz que Acalma | SIM (Sussurro) | +2 |
| Voz que Acalma | NÃO (ignora) | −2 |
| Herege | SIM (silencia) | −3 |
| Herege | NÃO (deixa falar) | +3 |
| Martírio | SIM (aceita) | −1 (Era 1–2) / +1 (Era 3) |
| Martírio | NÃO (protege) | +2 |

---

## 4. Dois Novos Dilemas

### Herege
**Trigger:** mesmo sistema aleatório existente (90–150s, sem modal ativo)
**Texto:** *"Um adepto questiona seus ensinamentos em público. O que fazer?"*

- **SIM — Silenciá-lo:** `Karma −3`, `Fe +8`, Adepto converte para Cético; flag `Herege_Silenciou = true`
- **NÃO — Deixá-lo falar:** `Karma +3`, `Fe −5`, Adepto ganha `Felicidade +20`, spawna 1 Cético na vizinhança; flag `Herege_Permitiu = true`

### Martírio
**Trigger:** mesmo sistema aleatório; só ocorre uma vez por jogo
**Texto:** *"Um adepto quer morrer pela fé. Aceitas o sacrifício?"*

- **SIM — Aceitar:** `Karma −1` (Eras 1–2) / `Karma +1` (Era 3), `Fe +15`, Adepto destruído, `Mortes +1`; flag `Martírio_Aceitou = true`
- **NÃO — Proibir:** `Karma +2`, `Fe −5`, Adepto `Felicidade +15`; flag `Martírio_Protegeu = true`

---

## 5. Flags para Final Secreto

Variáveis booleanas globais (iniciam em `false`, exceto onde indicado):

```
Justica_Perdoou        = false   (Justiça Divina → NÃO)
Voz_Acalmou            = false   (Voz que Acalma → SIM)
Herege_Permitiu        = false   (Herege → NÃO)
Martírio_Protegeu      = false   (Martírio → NÃO)
Martírio_Ocorreu       = false   (impede o dilema de disparar mais de uma vez)
Raio_Nunca_Adepto      = true    (inicia true; vira false se Raio atingir qualquer Adepto)
```

---

## 6. Os Cinco Finais

Avaliação ocorre quando a Era 3 é completada (botão "Evoluir" clicado na Era 3).

### Ordem de verificação (prioridade decrescente)

**1. Final Secreto — "A Encarnação"**
Condição: todas as 5 flags ativas + `Fe` entre 45–70

Antes do final normal, surge dilema especial único:
> *"Seu povo está inteiro. Mas sua presença os mantém presos à sua sombra. Você existe para eles, ou eles existem para você?"*
- SIM → Final Secreto
- NÃO → cai para Final Equilíbrio

**Texto do final secreto:**
> *"Você desceu sem anunciar. Viveu entre eles, morreu entre eles. Eles nunca souberam que foi você."*

---

**2. Fanatismo — "O Deus Vivo"**
Condição: `Fe > 75` ao completar Era 3

> *"Eles construíram uma estátua. Você não pediu e não consegue fazer eles pararem. Rezam para alguma versão de você que você não reconhece."*

---

**3. Ceticismo — "A Partida"**
Condição: `Fe < 45` ao completar Era 3

> *"Seu templo virou depósito. Ninguém entrou para quebrar nada, só pararam de aparecer. O povo está bem. Você que não está."*

---

**4. Tirania Divina — "O Senhor"**
Condição: `Fe` entre 45–75 + `Karma ≤ −3`

> *"Eles obedecem. Fazem festa no seu nome, cantam as músicas certas. Os olhos não acompanham."*

---

**5. Equilíbrio — "O Pastor"**
Condição: `Fe` entre 45–75 + `Karma > −3` (todos os outros casos)

> *"Eles acordam. Fizeram filhos, enterraram os mortos. Ainda acendem a vela. Não perguntam mais por você."*

---

## 7. Ajustes de Balanceamento

### Fanatismo

| Parâmetro | Atual | Proposto |
|-----------|-------|----------|
| Chance spawn Fanático | 35% / 6s | 20% / 8s |
| Cap de Fanáticos | `min(12, Adeptos/3)` | `min(8, Adeptos/4)` |
| Raio de dúvida (Cético base) | 100px | 80px |
| Raio de dúvida (estado Ceticismo) | 120px | 100px |

### Desastres

| Parâmetro | Atual | Proposto |
|-----------|-------|----------|
| Timer Era 2 | 60–90s | 90–130s |
| Dano de Fé por acerto Adepto | −10 | −7 |

### Progressão de Eras

| Meta | Atual | Proposto |
|------|-------|----------|
| Adeptos Era 1→2 | 30 | 25 |
| Adeptos Era 2→3 | 70 | 60 |
| Adeptos conclusão Era 3 | 150 | 120 |

### Reprodução

| Parâmetro | Atual | Proposto |
|-----------|-------|----------|
| Chance de nascimento | 10% / 5s | 15% / 5s |
| Felicidade mínima | 75 | 70 |

---

## Prioridades de Implementação

### Fase 1 — Essencial (resolve o problema principal)
1. Zonas visuais da barra de Fé + avisos progressivos
2. Ajustes de balanceamento (números)
3. Variável `Karma` + acúmulo nos dilemas existentes
4. Poder **Bênção** (Era 1)
5. Poder **Silêncio** (Era 3)
6. Cinco finais + texto

### Fase 2 — Enriquecimento
7. Dilema **Herege** + flags
8. Dilema **Martírio** + flags
9. Flags para final secreto
10. Final secreto + dilema de Encarnação

### Fase 3 — Se houver tempo
11. Poder **Redenção** (Era 2)
12. Poder **Chama** (Era 3)
