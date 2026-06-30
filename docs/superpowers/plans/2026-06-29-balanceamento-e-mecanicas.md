# Balanceamento e Novas Mecânicas — Plano de Implementação

> **Para agentes:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recomendado) ou superpowers:executing-plans para implementar este plano tarefa por tarefa. Steps usam checkbox (`- [ ]`) para rastreamento.

**Objetivo:** Implementar balanceamento, novos poderes (Bênção e Silêncio), karma, dois dilemas novos e cinco finais narrativos conforme spec `docs/superpowers/specs/2026-06-29-balanceamento-e-mecanicas-design.md`.

**Arquitetura:** Toda a lógica do jogo vive em `eventSheets/Folha de eventos 1.json`. Variáveis globais são definidas dentro do próprio event sheet como entradas `"eventType": "variable"`. Não há testes unitários — validação é JSON parse + MCP `validate_project` + teste manual no C3.

**Tech Stack:** Construct 3, edição JSON manual, MCP Construct 3

## Constraints Globais

- Nomes de variáveis e grupos: português brasileiro (exceto flags de código)
- Estados string exatos: `"Ceticismo"`, `"Equilibrado"`, `"Fanatismo"`
- SIDs novos: usar faixa `990000000000001`–`990000000000999` (únicos no projeto)
- JSON parse obrigatório antes de cada commit: `python -c "import json; json.load(open('eventSheets/Folha de eventos 1.json'))"`
- MCP `validate_project` após cada task completa
- Não recriar objetos removidos: BalaoOracao, sf_Pedido, MarcadorOracao, btn_Atender, btn_Negar

---

## Estrutura de Arquivos

| Arquivo | O que muda |
|---------|-----------|
| `eventSheets/Folha de eventos 1.json` | Todas as tasks — variáveis, eventos, ações |
| `project.c3proj` | Nenhuma mudança necessária (variáveis ficam no event sheet) |
| `layouts/Layout 1.json` | Apenas se novos objetos de UI precisarem ser posicionados |

---

## Task 1: Variáveis Globais

**Arquivo:** `eventSheets/Folha de eventos 1.json`

**Interfaces:**
- Produz: `Karma`, `Justica_Perdoou`, `Voz_Acalmou`, `Herege_Permitiu`, `Martírio_Protegeu`, `Martírio_Ocorreu`, `Raio_Nunca_Adepto` — usados em Tasks 4, 5, 6, 8, 9, 10

- [ ] **Step 1: Localizar o bloco de variáveis globais existente**

No arquivo `eventSheets/Folha de eventos 1.json`, buscar pelo SID `394863749690085` (variável `Fe`). As variáveis globais ficam no array raiz `"events"` como entradas `"eventType": "variable"`. Inserir as novas variáveis logo após a variável `Fe`.

- [ ] **Step 2: Inserir as 7 novas variáveis**

Adicionar ao array `"events"`, após a entrada da variável `Fe` (SID `394863749690085`):

```json
{
  "eventType": "variable",
  "name": "Karma",
  "type": "number",
  "initialValue": "0",
  "sid": 990000000000001
},
{
  "eventType": "variable",
  "name": "Justica_Perdoou",
  "type": "number",
  "initialValue": "0",
  "sid": 990000000000002
},
{
  "eventType": "variable",
  "name": "Voz_Acalmou",
  "type": "number",
  "initialValue": "0",
  "sid": 990000000000003
},
{
  "eventType": "variable",
  "name": "Herege_Permitiu",
  "type": "number",
  "initialValue": "0",
  "sid": 990000000000004
},
{
  "eventType": "variable",
  "name": "Martírio_Protegeu",
  "type": "number",
  "initialValue": "0",
  "sid": 990000000000005
},
{
  "eventType": "variable",
  "name": "Martírio_Ocorreu",
  "type": "number",
  "initialValue": "0",
  "sid": 990000000000006
},
{
  "eventType": "variable",
  "name": "Raio_Nunca_Adepto",
  "type": "number",
  "initialValue": "1",
  "sid": 990000000000007
},
{
  "eventType": "variable",
  "name": "Final_Definido",
  "type": "number",
  "initialValue": "0",
  "sid": 990000000000008
}
```

- [ ] **Step 3: Validar JSON**

```bash
python -c "import json; json.load(open('eventSheets/Folha de eventos 1.json')); print('OK')"
```

Esperado: `OK`

- [ ] **Step 4: Commit**

```bash
git add "eventSheets/Folha de eventos 1.json"
git commit -m "feat: adicionar variáveis globais Karma e flags de final secreto"
```

---

## Task 2: Ajustes de Balanceamento

**Arquivo:** `eventSheets/Folha de eventos 1.json`

Todos os valores abaixo são alterações de números em eventos existentes. Buscar pelo valor atual e substituir pelo proposto. Fazer as substituições uma de cada vez com verificação.

- [ ] **Step 1: Reduzir taxa de spawn de Fanáticos**

Buscar `"0.35"` na condição de chance de spawn de Fanático (próximo ao texto `"Fanatismo"` e timer de 6s). Alterar para `"0.20"`.

Buscar o timer de spawn do Fanático (tag `"SpawnFanatico"` ou similar, duração `"6"`). Alterar duração para `"8"`.

- [ ] **Step 2: Reduzir cap de Fanáticos**

Buscar a expressão `min(12, ceil(Porri_Adepto.Count / 3))`. Alterar para `min(8, ceil(Porri_Adepto.Count / 4))`.

- [ ] **Step 3: Reduzir alcance de dúvida dos Céticos**

Buscar variável `Cetico_Alcance` (SID `779390467503149`) — `"initialValue": "100"`. Alterar para `"80"`.

Buscar onde `Cetico_Alcance` é sobrescrito para `"120"` (quando Estado = "Ceticismo"). Alterar para `"100"`.

Buscar o valor de dano de dúvida `"4"` (Cético drena Felicidade). Alterar para `"3"`.
Buscar o valor de dano no estado Ceticismo `"6"`. Alterar para `"5"`.

- [ ] **Step 4: Reduzir frequência e dano dos desastres**

Buscar o timer `"Desastre"` com `random(60,90)`. Alterar para `random(90,130)`.

Buscar a ação de dano de Fé no evento de colisão do Furacão com Adepto (`Fe - 10` ou `set Fe to Fe-10`). Alterar de `10` para `7`.

- [ ] **Step 5: Ajustar metas de progressão de Eras**

Buscar a condição `Porri_Adepto.Count >= 30` (meta Era 1). Alterar para `>= 25`.

Buscar a condição `Porri_Adepto.Count >= 70` (meta Era 2). Alterar para `>= 60`.

Buscar a condição `Porri_Adepto.Count >= 150` (meta Era 3). Alterar para `>= 120`.

Buscar os textos de objetivo que exibem "30", "70", "150" em `txt_Objetivos`. Atualizar para "25", "60", "120" respectivamente.

- [ ] **Step 6: Aumentar taxa de reprodução**

Buscar a condição de nascimento com `"second-value": "0.1"` (SID `967126181250303`). Alterar para `"0.15"`.

Buscar a condição de Felicidade mínima para reprodução (`Felicidade >= 75`). Alterar para `>= 70`.

- [ ] **Step 7: Validar e commitar**

```bash
python -c "import json; json.load(open('eventSheets/Folha de eventos 1.json')); print('OK')"
```

```bash
git add "eventSheets/Folha de eventos 1.json"
git commit -m "balance: ajustar taxas de fanatismo, desastre, reprodução e metas de era"
```

---

## Task 3: Zonas Visuais de Fé e Alertas

**Arquivo:** `eventSheets/Folha de eventos 1.json`

Adiciona feedback visual da barra de Fé com zonas de cor e avisos textuais progressivos.

**Interfaces:**
- Consome: variável `Fe`, objeto `ponteiro_Fe`, objeto `Txt_Estado`
- Produz: coloração da barra de Fe + textos de alerta visíveis ao jogador

- [ ] **Step 1: Localizar o grupo de UI "Every tick"**

Buscar o grupo de eventos que atualiza o HUD a cada tick (contém ações em `ponteiro_Fe`, `Txt_Estado`, `txt_Era`). Inserir ao final desse grupo os eventos de zona de cor.

- [ ] **Step 2: Adicionar evento de zona Verde (Equilibrado)**

Adicionar bloco ao grupo de UI:

```json
{
  "eventType": "block",
  "sid": 990000000000010,
  "conditions": [
    {
      "id": "compare-two-values",
      "objectClass": "System",
      "sid": 990000000000011,
      "parameters": {
        "first-value": "Fe",
        "comparison": 5,
        "second-value": "45"
      }
    },
    {
      "id": "compare-two-values",
      "objectClass": "System",
      "sid": 990000000000012,
      "parameters": {
        "first-value": "Fe",
        "comparison": 2,
        "second-value": "66"
      }
    }
  ],
  "actions": [
    {
      "id": "set-color",
      "objectClass": "ponteiro_Fe",
      "sid": 990000000000013,
      "parameters": {
        "color": "rgbEx(60, 180, 60)"
      }
    }
  ]
}
```

*(comparison 5 = >=, comparison 2 = <)*

- [ ] **Step 3: Adicionar evento de zona Laranja (Atenção alta, Fe 66–79)**

```json
{
  "eventType": "block",
  "sid": 990000000000014,
  "conditions": [
    {
      "id": "compare-two-values",
      "objectClass": "System",
      "sid": 990000000000015,
      "parameters": {
        "first-value": "Fe",
        "comparison": 5,
        "second-value": "66"
      }
    },
    {
      "id": "compare-two-values",
      "objectClass": "System",
      "sid": 990000000000016,
      "parameters": {
        "first-value": "Fe",
        "comparison": 2,
        "second-value": "80"
      }
    }
  ],
  "actions": [
    {
      "id": "set-color",
      "objectClass": "ponteiro_Fe",
      "sid": 990000000000017,
      "parameters": {
        "color": "rgbEx(220, 140, 30)"
      }
    },
    {
      "id": "set-text",
      "objectClass": "Txt_Estado",
      "sid": 990000000000018,
      "parameters": {
        "text": "\"⚠ FÉ ELEVADA\""
      }
    }
  ]
}
```

- [ ] **Step 4: Adicionar evento de zona Vermelha (Fanatismo, Fe >= 80)**

```json
{
  "eventType": "block",
  "sid": 990000000000019,
  "conditions": [
    {
      "id": "compare-two-values",
      "objectClass": "System",
      "sid": 990000000000020,
      "parameters": {
        "first-value": "Fe",
        "comparison": 5,
        "second-value": "80"
      }
    }
  ],
  "actions": [
    {
      "id": "set-color",
      "objectClass": "ponteiro_Fe",
      "sid": 990000000000021,
      "parameters": {
        "color": "rgbEx(210, 40, 40)"
      }
    },
    {
      "id": "set-text",
      "objectClass": "Txt_Estado",
      "sid": 990000000000022,
      "parameters": {
        "text": "\"O POVO ESTÁ EXALTADO\""
      }
    }
  ]
}
```

- [ ] **Step 5: Adicionar evento de zona Azul (Ceticismo, Fe < 31)**

```json
{
  "eventType": "block",
  "sid": 990000000000023,
  "conditions": [
    {
      "id": "compare-two-values",
      "objectClass": "System",
      "sid": 990000000000024,
      "parameters": {
        "first-value": "Fe",
        "comparison": 2,
        "second-value": "31"
      }
    }
  ],
  "actions": [
    {
      "id": "set-color",
      "objectClass": "ponteiro_Fe",
      "sid": 990000000000025,
      "parameters": {
        "color": "rgbEx(60, 100, 200)"
      }
    },
    {
      "id": "set-text",
      "objectClass": "Txt_Estado",
      "sid": 990000000000026,
      "parameters": {
        "text": "\"A FÉ ENFRAQUECE\""
      }
    }
  ]
}
```

- [ ] **Step 6: Marcar Raio_Nunca_Adepto = 0 quando Raio acerta Adepto**

Localizar o evento de colisão `Raio` com `Porri_Adepto`. Adicionar ação ao final do bloco:

```json
{
  "id": "set-eventvar-value",
  "objectClass": "System",
  "sid": 990000000000027,
  "parameters": {
    "variable": "Raio_Nunca_Adepto",
    "value": "0"
  }
}
```

- [ ] **Step 7: Validar e commitar**

```bash
python -c "import json; json.load(open('eventSheets/Folha de eventos 1.json')); print('OK')"
git add "eventSheets/Folha de eventos 1.json"
git commit -m "feat: adicionar zonas visuais de fé e alertas de estado"
```

**Teste manual:** Abrir no C3, verificar que a barra de Fe muda de cor ao aproximar-se dos thresholds. Ajustar os valores `rgbEx` se as cores não aparecerem (checar se o objeto `ponteiro_Fe` suporta `set-color` — se não, usar `set-blend-color` com parâmetros r, g, b separados).

---

## Task 4: Karma nos Dilemas Existentes

**Arquivo:** `eventSheets/Folha de eventos 1.json`

Adiciona delta de Karma aos handlers SIM e NÃO dos três dilemas existentes.

**Interfaces:**
- Consome: SID do btn_Dilema_Sim handler `811619735514069`, variável `Karma`
- Produz: `Karma` acumulado para uso em Task 7; flags `Justica_Perdoou` e `Voz_Acalmou`

- [ ] **Step 1: Adicionar Karma ao handler SIM de Festa_Esperanca (SID filho: `559479576955054`)**

No child block de Festa_Esperanca dentro do btn_Dilema_Sim handler, adicionar ação:

```json
{
  "id": "set-eventvar-value",
  "objectClass": "System",
  "sid": 990000000000030,
  "parameters": {
    "variable": "Karma",
    "value": "Karma + 2"
  }
}
```

- [ ] **Step 2: Adicionar Karma ao handler NÃO de Festa_Esperanca**

No child block de Festa_Esperanca dentro do btn_Dilema_Nao handler, adicionar:

```json
{
  "id": "set-eventvar-value",
  "objectClass": "System",
  "sid": 990000000000031,
  "parameters": {
    "variable": "Karma",
    "value": "Karma - 2"
  }
}
```

- [ ] **Step 3: Adicionar Karma + flag ao handler SIM de Justica_Vizinho (perdoa)**

O SIM de Justica_Vizinho é a ação de Raio. Localizar o child block de Justica_Vizinho no btn_Dilema_Nao (NÃO = perdoar = SIM do karma positivo). Adicionar:

```json
{
  "id": "set-eventvar-value",
  "objectClass": "System",
  "sid": 990000000000032,
  "parameters": {
    "variable": "Karma",
    "value": "Karma + 3"
  }
},
{
  "id": "set-eventvar-value",
  "objectClass": "System",
  "sid": 990000000000033,
  "parameters": {
    "variable": "Justica_Perdoou",
    "value": "1"
  }
}
```

- [ ] **Step 4: Adicionar Karma ao handler SIM de Justica_Vizinho (castiga)**

No child block SIM de Justica_Vizinho (usa Raio), adicionar:

```json
{
  "id": "set-eventvar-value",
  "objectClass": "System",
  "sid": 990000000000034,
  "parameters": {
    "variable": "Karma",
    "value": "Karma - 3"
  }
}
```

- [ ] **Step 5: Adicionar Karma + flag ao handler SIM de Voz_Acalma (Sussurro)**

No child block SIM de Voz_Acalma (SID filho: `757957590032529`), adicionar:

```json
{
  "id": "set-eventvar-value",
  "objectClass": "System",
  "sid": 990000000000035,
  "parameters": {
    "variable": "Karma",
    "value": "Karma + 2"
  }
},
{
  "id": "set-eventvar-value",
  "objectClass": "System",
  "sid": 990000000000036,
  "parameters": {
    "variable": "Voz_Acalmou",
    "value": "1"
  }
}
```

- [ ] **Step 6: Adicionar Karma ao handler NÃO de Voz_Acalma**

No child block NÃO de Voz_Acalma, adicionar:

```json
{
  "id": "set-eventvar-value",
  "objectClass": "System",
  "sid": 990000000000037,
  "parameters": {
    "variable": "Karma",
    "value": "Karma - 2"
  }
}
```

- [ ] **Step 7: Validar e commitar**

```bash
python -c "import json; json.load(open('eventSheets/Folha de eventos 1.json')); print('OK')"
git add "eventSheets/Folha de eventos 1.json"
git commit -m "feat: adicionar sistema de karma nos dilemas existentes"
```

---

## Task 5: Poder Bênção

**Arquivo:** `eventSheets/Folha de eventos 1.json`

**Interfaces:**
- Consome: variáveis `Poderes_IDs` (SID `920000000000026`), `Poderes_Animacoes` (SID `920000000000025`), `Poderes_Eras` (SID `920000000000027`), `Poderes_Cooldowns` (SID `920000000000028`); objetos `Porri_Adepto`, `btn_Poderes`, `Poder_Selecionado`
- Produz: poder "Bencao" selecionável e funcional

- [ ] **Step 1: Adicionar animação "Bencao" ao btn_Poderes no C3 Editor**

Abrir o projeto no Construct 3. No objeto `btn_Poderes`, adicionar uma nova animação chamada `"Bencao"`. Usar o mesmo frame de `"Sussurro"` como placeholder (copiar a animação). Salvar e fechar o editor.

- [ ] **Step 2: Atualizar a variável Poderes_IDs**

Localizar a variável `Poderes_IDs` (SID `920000000000026`). Alterar `"initialValue"`:

De: `"Raio|Sussurro"`
Para: `"Raio|Bencao|Sussurro"`

- [ ] **Step 3: Atualizar Poderes_Animacoes**

Localizar SID `920000000000025`. Alterar:

De: `"Relampago|Sussurro"`
Para: `"Relampago|Bencao|Sussurro"`

- [ ] **Step 4: Atualizar Poderes_Eras**

Localizar SID `920000000000027`. Alterar:

De: `"1|1"`
Para: `"1|1|2"`

*(Raio=Era1, Bênção=Era1, Sussurro=Era2)*

- [ ] **Step 5: Atualizar Poderes_Cooldowns**

Localizar SID `920000000000028`. Alterar:

De: `"4|7"`
Para: `"4|8|7"`

*(Raio=4s, Bênção=8s, Sussurro=7s)*

- [ ] **Step 6: Adicionar handler de uso da Bênção**

Localizar o grupo "Poderes" no event sheet. Após o handler de uso do Sussurro (clique em Porri_Cetico quando Poder_Selecionado = "Sussurro"), adicionar o novo bloco:

```json
{
  "eventType": "block",
  "sid": 990000000000040,
  "conditions": [
    {
      "id": "on-clicked",
      "objectClass": "Porri_Adepto",
      "sid": 990000000000041,
      "parameters": {
        "mouse-button": 0
      }
    },
    {
      "id": "compare-two-values",
      "objectClass": "System",
      "sid": 990000000000042,
      "parameters": {
        "first-value": "Poder_Selecionado",
        "comparison": 0,
        "second-value": "\"Bencao\""
      }
    }
  ],
  "actions": [
    {
      "id": "set-instvar-value",
      "objectClass": "Porri_Adepto",
      "sid": 990000000000043,
      "parameters": {
        "instance-variable": "Felicidade",
        "value": "min(100, Porri_Adepto.Felicidade + 25)"
      }
    },
    {
      "id": "set-eventvar-value",
      "objectClass": "System",
      "sid": 990000000000044,
      "parameters": {
        "variable": "Poder_Selecionado",
        "value": "\"\""
      }
    }
  ]
}
```

*(Nota: o cooldown é aplicado pelo sistema existente de btn_Poderes ao detectar uso do poder — checar se é necessário disparar o timer de CD manualmente ou se o sistema já gerencia isso ao limpar Poder_Selecionado.)*

- [ ] **Step 7: Validar e commitar**

```bash
python -c "import json; json.load(open('eventSheets/Folha de eventos 1.json')); print('OK')"
git add "eventSheets/Folha de eventos 1.json"
git commit -m "feat: adicionar poder Bênção (Era 1, CD 8s)"
```

**Teste manual:** Iniciar o jogo, verificar se o botão de Bênção aparece na Era 1. Clicar no botão e depois num Adepto — verificar que a Felicidade sobe 25 pontos.

---

## Task 6: Poder Silêncio

**Arquivo:** `eventSheets/Folha de eventos 1.json`

**Interfaces:**
- Consome: mesmas variáveis de poder da Task 5; variável `Fe`; instância `Porri_Adepto.Imune`
- Produz: poder "Silencio" — reduz Fe em 12, imuniza Adeptos por 3s

- [ ] **Step 1: Adicionar animação "Silencio" ao btn_Poderes no C3 Editor**

No Construct 3, adicionar animação `"Silencio"` ao `btn_Poderes`. Usar frame de `"Relampago"` como placeholder com tint branco.

- [ ] **Step 2: Atualizar variáveis de poder para incluir Silêncio (Era 3)**

Alterar `Poderes_IDs` (SID `920000000000026`):

De: `"Raio|Bencao|Sussurro"`
Para: `"Raio|Bencao|Sussurro|Silencio"`

Alterar `Poderes_Animacoes` (SID `920000000000025`):

De: `"Relampago|Bencao|Sussurro"`
Para: `"Relampago|Bencao|Sussurro|Silencio"`

Alterar `Poderes_Eras` (SID `920000000000027`):

De: `"1|1|2"`
Para: `"1|1|2|3"`

Alterar `Poderes_Cooldowns` (SID `920000000000028`):

De: `"4|8|7"`
Para: `"4|8|7|15"`

- [ ] **Step 3: Adicionar handler de uso do Silêncio**

No grupo "Poderes", adicionar após o handler da Bênção:

```json
{
  "eventType": "block",
  "sid": 990000000000050,
  "conditions": [
    {
      "id": "on-mouse-button-pressed",
      "objectClass": "Mouse",
      "sid": 990000000000051,
      "parameters": {
        "mouse-button": 0
      }
    },
    {
      "id": "compare-two-values",
      "objectClass": "System",
      "sid": 990000000000052,
      "parameters": {
        "first-value": "Poder_Selecionado",
        "comparison": 0,
        "second-value": "\"Silencio\""
      }
    }
  ],
  "actions": [
    {
      "id": "set-eventvar-value",
      "objectClass": "System",
      "sid": 990000000000053,
      "parameters": {
        "variable": "Fe",
        "value": "max(0, Fe - 12)"
      }
    },
    {
      "id": "set-instvar-value",
      "objectClass": "Porri_Adepto",
      "sid": 990000000000054,
      "parameters": {
        "instance-variable": "Imune",
        "value": "1"
      }
    },
    {
      "id": "set-eventvar-value",
      "objectClass": "System",
      "sid": 990000000000055,
      "parameters": {
        "variable": "Poder_Selecionado",
        "value": "\"\""
      }
    }
  ]
}
```

- [ ] **Step 4: Adicionar timer para remover imunidade após 3s**

Ainda dentro do mesmo bloco, após a ação de setar `Imune = 1`, adicionar ação de timer:

```json
{
  "id": "start-timer",
  "objectClass": "Controlador",
  "sid": 990000000000056,
  "behaviorType": "Cronômetro",
  "parameters": {
    "duration": "3",
    "type": "once",
    "tag": "\"RemoverImunidade\""
  }
}
```

Adicionar evento de timer "RemoverImunidade" no grupo de setup ou no grupo de timers existente:

```json
{
  "eventType": "block",
  "sid": 990000000000057,
  "conditions": [
    {
      "id": "on-timer",
      "objectClass": "Controlador",
      "sid": 990000000000058,
      "behaviorType": "Cronômetro",
      "parameters": {
        "tag": "\"RemoverImunidade\""
      }
    }
  ],
  "actions": [
    {
      "id": "set-instvar-value",
      "objectClass": "Porri_Adepto",
      "sid": 990000000000059,
      "parameters": {
        "instance-variable": "Imune",
        "value": "0"
      }
    }
  ]
}
```

- [ ] **Step 5: Validar e commitar**

```bash
python -c "import json; json.load(open('eventSheets/Folha de eventos 1.json')); print('OK')"
git add "eventSheets/Folha de eventos 1.json"
git commit -m "feat: adicionar poder Silêncio (Era 3, CD 15s)"
```

**Teste manual:** Na Era 3, selecionar Silêncio, clicar na tela — verificar que Fe cai 12 pontos e os Adeptos ficam imunes a dúvida por 3s.

---

## Task 7: Sistema de Finais

**Arquivo:** `eventSheets/Folha de eventos 1.json`

Avalia condições ao completar Era 3 e exibe o final correspondente usando os objetos de dilema existentes (sem criar nova UI).

**Interfaces:**
- Consome: `Fe`, `Karma`, `Era`, `btn_Evoluir`, objetos `Filtro_Dilema`, `Painel_Dilema`, `txt_Dilema_Titulo`, `txt_Dilema_Corpo`, `txt_Dilema_Efeitos`, `btn_Dilema_Sim`, `btn_Dilema_Nao`
- Produz: tela de final exibida quando Era == 3 e jogador clica em Evoluir

- [ ] **Step 1: Localizar o evento de clique em btn_Evoluir**

Buscar pelo event handler de `btn_Evoluir` no event sheet. Atualmente ele incrementa `Era`. Vamos adicionar um bloco filho ANTES do incremento que verifica se Era já é 3.

- [ ] **Step 2: Adicionar verificação de Era 3 antes do incremento**

No evento de clique em `btn_Evoluir`, adicionar bloco filho com condição `Era = 3`:

```json
{
  "eventType": "block",
  "sid": 990000000000060,
  "conditions": [
    {
      "id": "compare-two-values",
      "objectClass": "System",
      "sid": 990000000000061,
      "parameters": {
        "first-value": "Era",
        "comparison": 0,
        "second-value": "3"
      }
    }
  ],
  "actions": [
    {
      "id": "set-eventvar-value",
      "objectClass": "System",
      "sid": 990000000000062,
      "parameters": {
        "variable": "Dilema_Ativo",
        "value": "1"
      }
    },
    {
      "id": "set-time-scale",
      "objectClass": "System",
      "sid": 990000000000063,
      "parameters": {
        "time-scale": "0"
      }
    },
    {
      "id": "set-visible",
      "objectClass": "Filtro_Dilema",
      "sid": 990000000000064,
      "parameters": {
        "visibility": "visible"
      }
    },
    {
      "id": "set-visible",
      "objectClass": "Painel_Dilema",
      "sid": 990000000000065,
      "parameters": {
        "visibility": "visible"
      }
    },
    {
      "id": "set-visible",
      "objectClass": "btn_Dilema_Sim",
      "sid": 990000000000066,
      "parameters": {
        "visibility": "invisible"
      }
    },
    {
      "id": "set-visible",
      "objectClass": "btn_Dilema_Nao",
      "sid": 990000000000067,
      "parameters": {
        "visibility": "invisible"
      }
    },
    {
      "id": "set-eventvar-value",
      "objectClass": "System",
      "sid": 990000000000068,
      "parameters": {
        "variable": "Dilema_Tipo",
        "value": "\"Final\""
      }
    }
  ],
  "children": [
    {
      "eventType": "block",
      "sid": 990000000000070,
      "conditions": [
        {
          "id": "compare-two-values",
          "objectClass": "System",
          "sid": 990000000000071,
          "parameters": {
            "first-value": "Fe",
            "comparison": 3,
            "second-value": "75"
          }
        }
      ],
      "actions": [
        {
          "id": "set-text",
          "objectClass": "txt_Dilema_Titulo",
          "sid": 990000000000072,
          "parameters": {
            "text": "\"O DEUS VIVO\""
          }
        },
        {
          "id": "set-text",
          "objectClass": "txt_Dilema_Corpo",
          "sid": 990000000000073,
          "parameters": {
            "text": "\"Eles construíram uma estátua.\" & newline & \"Você não pediu e não\" & newline & \"consegue fazer eles\" & newline & \"pararem. Rezam para\" & newline & \"alguma versão de você\" & newline & \"que você não reconhece.\""
          }
        },
        {
          "id": "set-text",
          "objectClass": "txt_Dilema_Efeitos",
          "sid": 990000000000074,
          "parameters": {
            "text": "\"\""
          }
        }
      ]
    },
    {
      "eventType": "block",
      "sid": 990000000000075,
      "conditions": [
        {
          "id": "compare-two-values",
          "objectClass": "System",
          "sid": 990000000000076,
          "parameters": {
            "first-value": "Fe",
            "comparison": 2,
            "second-value": "45"
          }
        }
      ],
      "actions": [
        {
          "id": "set-text",
          "objectClass": "txt_Dilema_Titulo",
          "sid": 990000000000077,
          "parameters": {
            "text": "\"A PARTIDA\""
          }
        },
        {
          "id": "set-text",
          "objectClass": "txt_Dilema_Corpo",
          "sid": 990000000000078,
          "parameters": {
            "text": "\"Seu templo virou depósito.\" & newline & \"Ninguém entrou para\" & newline & \"quebrar nada, só pararam\" & newline & \"de aparecer. O povo\" & newline & \"está bem. Você que\" & newline & \"não está.\""
          }
        },
        {
          "id": "set-text",
          "objectClass": "txt_Dilema_Efeitos",
          "sid": 990000000000079,
          "parameters": {
            "text": "\"\""
          }
        }
      ]
    },
    {
      "eventType": "block",
      "sid": 990000000000080,
      "conditions": [
        {
          "id": "compare-two-values",
          "objectClass": "System",
          "sid": 990000000000081,
          "parameters": {
            "first-value": "Fe",
            "comparison": 5,
            "second-value": "45"
          }
        },
        {
          "id": "compare-two-values",
          "objectClass": "System",
          "sid": 990000000000082,
          "parameters": {
            "first-value": "Fe",
            "comparison": 4,
            "second-value": "75"
          }
        },
        {
          "id": "compare-two-values",
          "objectClass": "System",
          "sid": 990000000000083,
          "parameters": {
            "first-value": "Karma",
            "comparison": 4,
            "second-value": "-3"
          }
        }
      ],
      "actions": [
        {
          "id": "set-text",
          "objectClass": "txt_Dilema_Titulo",
          "sid": 990000000000084,
          "parameters": {
            "text": "\"O SENHOR\""
          }
        },
        {
          "id": "set-text",
          "objectClass": "txt_Dilema_Corpo",
          "sid": 990000000000085,
          "parameters": {
            "text": "\"Eles obedecem. Fazem\" & newline & \"festa no seu nome,\" & newline & \"cantam as músicas certas.\" & newline & \"Os olhos não acompanham.\""
          }
        },
        {
          "id": "set-text",
          "objectClass": "txt_Dilema_Efeitos",
          "sid": 990000000000086,
          "parameters": {
            "text": "\"\""
          }
        }
      ]
    },
    {
      "eventType": "block",
      "sid": 990000000000087,
      "conditions": [
        {
          "id": "compare-two-values",
          "objectClass": "System",
          "sid": 990000000000088,
          "parameters": {
            "first-value": "Fe",
            "comparison": 5,
            "second-value": "45"
          }
        },
        {
          "id": "compare-two-values",
          "objectClass": "System",
          "sid": 990000000000089,
          "parameters": {
            "first-value": "Fe",
            "comparison": 4,
            "second-value": "75"
          }
        },
        {
          "id": "compare-two-values",
          "objectClass": "System",
          "sid": 990000000000090,
          "parameters": {
            "first-value": "Karma",
            "comparison": 3,
            "second-value": "-3"
          }
        }
      ],
      "actions": [
        {
          "id": "set-text",
          "objectClass": "txt_Dilema_Titulo",
          "sid": 990000000000091,
          "parameters": {
            "text": "\"O PASTOR\""
          }
        },
        {
          "id": "set-text",
          "objectClass": "txt_Dilema_Corpo",
          "sid": 990000000000092,
          "parameters": {
            "text": "\"Eles acordam. Fizeram\" & newline & \"filhos, enterraram os mortos.\" & newline & \"Ainda acendem a vela.\" & newline & \"Não perguntam mais\" & newline & \"por você.\""
          }
        },
        {
          "id": "set-text",
          "objectClass": "txt_Dilema_Efeitos",
          "sid": 990000000000093,
          "parameters": {
            "text": "\"\""
          }
        }
      ]
    }
  ]
}
```

*(comparison 3 = >, comparison 4 = <=, comparison 5 = >=, comparison 2 = <)*

- [ ] **Step 3: Adicionar condição `Final_Definido = 0` em cada bloco de final normal**

Cada um dos quatro blocos de final (Fanatismo SID `990000000000070`, Ceticismo SID `990000000000075`, Tirania SID `990000000000080`, Equilíbrio SID `990000000000087`) deve ter a condição adicional:

```json
{
  "id": "compare-two-values",
  "objectClass": "System",
  "sid": 990000000000095,
  "parameters": {
    "first-value": "Final_Definido",
    "comparison": 0,
    "second-value": "0"
  }
}
```

*(Usar SIDs incrementais: `990000000000095`, `990000000000096`, `990000000000097`, `990000000000098` para cada bloco. Isso garante que o final secreto, ao setar `Final_Definido = 1`, impede os outros de sobrescrever o texto.)*

- [ ] **Step 4: Adicionar lógica de "stop" no handler atual de btn_Evoluir**

O handler atual de `btn_Evoluir` sempre incrementa `Era`. Envolver essa ação numa condição `Era < 3` para que na Era 3 ela não execute (o bloco de final já congelou o jogo):

Localizar a ação `Era = Era + 1` e adicionar condição pai: `Era < 3`.

- [ ] **Step 5: Validar e commitar**

```bash
python -c "import json; json.load(open('eventSheets/Folha de eventos 1.json')); print('OK')"
git add "eventSheets/Folha de eventos 1.json"
git commit -m "feat: adicionar sistema de finais narrativos (4 finais)"
```

**Teste manual:** Cheat a Era para 3 (via debugger C3), atender os objetivos e clicar em Evoluir. Verificar que o painel de final aparece com o texto correto baseado em Fe e Karma.

---

## Task 8 (Fase 2): Dilema Herege

**Arquivo:** `eventSheets/Folha de eventos 1.json`

**Interfaces:**
- Consome: sistema de dilemas existente, SID de Dilema_Tipo setter, `Karma`, flag `Herege_Permitiu`
- Produz: dilema "Herege" funcional com karma e flag

- [ ] **Step 1: Adicionar "Herege" ao pool de dilemas**

Localizar o evento de set `Dilema_Tipo` (expressão com `choose(...)`). Alterar:

De:
```
Porri_Adepto.Count < 2 ? "Festa_Esperanca" : choose("Festa_Esperanca", "Justica_Vizinho", "Voz_Acalma")
```

Para:
```
Porri_Adepto.Count < 2 ? "Festa_Esperanca" : choose("Festa_Esperanca", "Justica_Vizinho", "Voz_Acalma", "Herege")
```

- [ ] **Step 2: Adicionar bloco de exibição do Herege**

No grupo de exibição de dilemas (onde Festa_Esperanca, Justica_Vizinho e Voz_Acalma são exibidos), adicionar bloco:

```json
{
  "eventType": "block",
  "sid": 990000000000100,
  "conditions": [
    {
      "id": "compare-two-values",
      "objectClass": "System",
      "sid": 990000000000101,
      "parameters": {
        "first-value": "Dilema_Tipo",
        "comparison": 0,
        "second-value": "\"Herege\""
      }
    }
  ],
  "actions": [
    {
      "id": "set-text",
      "objectClass": "txt_Dilema_Titulo",
      "sid": 990000000000102,
      "parameters": {
        "text": "\"HERESIA\""
      }
    },
    {
      "id": "set-text",
      "objectClass": "txt_Dilema_Corpo",
      "sid": 990000000000103,
      "parameters": {
        "text": "\"ELE ESTAVA ME PREGANDO\" & newline & \"CONTRA EM VOZ ALTA.\" & newline & \"ISSO É PERMITIDO?\""
      }
    },
    {
      "id": "set-text",
      "objectClass": "txt_Dilema_Efeitos",
      "sid": 990000000000104,
      "parameters": {
        "text": "\"SIM SILENCIAR\" & newline & \"-3 KARMA\" & newline & \"+8 FÉ\" & newline & \"ADEPTO VIRA CÉTICO\" & newline & newline & \"NÃO DEIXAR FALAR\" & newline & \"+3 KARMA\" & newline & \"-5 FÉ\" & newline & \"+20 FELICIDADE\""
      }
    }
  ]
}
```

- [ ] **Step 3: Adicionar handler SIM do Herege (silenciar)**

No bloco do btn_Dilema_Sim, adicionar child block:

```json
{
  "eventType": "block",
  "sid": 990000000000105,
  "conditions": [
    {
      "id": "compare-two-values",
      "objectClass": "System",
      "sid": 990000000000106,
      "parameters": {
        "first-value": "Dilema_Tipo",
        "comparison": 0,
        "second-value": "\"Herege\""
      }
    }
  ],
  "actions": [
    {
      "id": "set-eventvar-value",
      "objectClass": "System",
      "sid": 990000000000107,
      "parameters": {
        "variable": "Karma",
        "value": "Karma - 3"
      }
    },
    {
      "id": "set-eventvar-value",
      "objectClass": "System",
      "sid": 990000000000108,
      "parameters": {
        "variable": "Fe",
        "value": "min(100, Fe + 8)"
      }
    }
  ]
}
```

*(A conversão do Adepto "dono" do dilema para Cético segue o mesmo padrão de Justica_Vizinho — usar `Dilema_Dono_UID` para encontrar a instância e destruí-la, criando um Porri_Cetico na mesma posição.)*

- [ ] **Step 4: Adicionar handler NÃO do Herege (deixar falar) + flag**

No bloco do btn_Dilema_Nao, adicionar child block:

```json
{
  "eventType": "block",
  "sid": 990000000000109,
  "conditions": [
    {
      "id": "compare-two-values",
      "objectClass": "System",
      "sid": 990000000000110,
      "parameters": {
        "first-value": "Dilema_Tipo",
        "comparison": 0,
        "second-value": "\"Herege\""
      }
    }
  ],
  "actions": [
    {
      "id": "set-eventvar-value",
      "objectClass": "System",
      "sid": 990000000000111,
      "parameters": {
        "variable": "Karma",
        "value": "Karma + 3"
      }
    },
    {
      "id": "set-eventvar-value",
      "objectClass": "System",
      "sid": 990000000000112,
      "parameters": {
        "variable": "Fe",
        "value": "max(0, Fe - 5)"
      }
    },
    {
      "id": "set-eventvar-value",
      "objectClass": "System",
      "sid": 990000000000113,
      "parameters": {
        "variable": "Herege_Permitiu",
        "value": "1"
      }
    }
  ]
}
```

*(Adicionar também +20 Felicidade ao Adepto `Dilema_Dono_UID` — usar `pick-by-unique-id` para selecionar a instância e `set-instvar-value` em Felicidade.)*

- [ ] **Step 5: Validar e commitar**

```bash
python -c "import json; json.load(open('eventSheets/Folha de eventos 1.json')); print('OK')"
git add "eventSheets/Folha de eventos 1.json"
git commit -m "feat: adicionar dilema Herege com karma e flag"
```

---

## Task 9 (Fase 2): Dilema Martírio

**Arquivo:** `eventSheets/Folha de eventos 1.json`

Dilema único (ocorre no máximo uma vez por jogo).

- [ ] **Step 1: Adicionar "Martírio" ao pool de dilemas (condicional)**

Localizar o evento de set `Dilema_Tipo`. Alterar a expressão para incluir Martírio apenas quando `Martírio_Ocorreu = 0`:

```
Porri_Adepto.Count < 2 ? "Festa_Esperanca" : (Martírio_Ocorreu = 0 ? choose("Festa_Esperanca", "Justica_Vizinho", "Voz_Acalma", "Herege", "Martírio") : choose("Festa_Esperanca", "Justica_Vizinho", "Voz_Acalma", "Herege"))
```

- [ ] **Step 2: Adicionar bloco de exibição do Martírio**

```json
{
  "eventType": "block",
  "sid": 990000000000120,
  "conditions": [
    {
      "id": "compare-two-values",
      "objectClass": "System",
      "sid": 990000000000121,
      "parameters": {
        "first-value": "Dilema_Tipo",
        "comparison": 0,
        "second-value": "\"Martírio\""
      }
    }
  ],
  "actions": [
    {
      "id": "set-text",
      "objectClass": "txt_Dilema_Titulo",
      "sid": 990000000000122,
      "parameters": {
        "text": "\"MARTÍRIO\""
      }
    },
    {
      "id": "set-text",
      "objectClass": "txt_Dilema_Corpo",
      "sid": 990000000000123,
      "parameters": {
        "text": "\"ESTOU PRONTO PARA\" & newline & \"O MARTÍRIO. ABENÇOAS?\""
      }
    },
    {
      "id": "set-text",
      "objectClass": "txt_Dilema_Efeitos",
      "sid": 990000000000124,
      "parameters": {
        "text": "\"SIM ACEITAR\" & newline & \"+15 FÉ\" & newline & \"-1 ADEPTO\" & newline & newline & \"NÃO PROIBIR\" & newline & \"+2 KARMA\" & newline & \"-5 FÉ\" & newline & \"+15 FELICIDADE\""
      }
    }
  ]
}
```

- [ ] **Step 3: Adicionar handler SIM do Martírio**

```json
{
  "eventType": "block",
  "sid": 990000000000125,
  "conditions": [
    {
      "id": "compare-two-values",
      "objectClass": "System",
      "sid": 990000000000126,
      "parameters": {
        "first-value": "Dilema_Tipo",
        "comparison": 0,
        "second-value": "\"Martírio\""
      }
    }
  ],
  "actions": [
    {
      "id": "set-eventvar-value",
      "objectClass": "System",
      "sid": 990000000000127,
      "parameters": {
        "variable": "Fe",
        "value": "min(100, Fe + 15)"
      }
    },
    {
      "id": "set-eventvar-value",
      "objectClass": "System",
      "sid": 990000000000128,
      "parameters": {
        "variable": "Mortes",
        "value": "Mortes + 1"
      }
    },
    {
      "id": "set-eventvar-value",
      "objectClass": "System",
      "sid": 990000000000129,
      "parameters": {
        "variable": "Karma",
        "value": "Era >= 3 ? Karma + 1 : Karma - 1"
      }
    },
    {
      "id": "set-eventvar-value",
      "objectClass": "System",
      "sid": 990000000000130,
      "parameters": {
        "variable": "Martírio_Ocorreu",
        "value": "1"
      }
    }
  ]
}
```

*(Adicionar também destruição do Adepto `Dilema_Dono_UID` usando pick-by-unique-id + destroy.)*

- [ ] **Step 4: Adicionar handler NÃO do Martírio + flag**

```json
{
  "eventType": "block",
  "sid": 990000000000131,
  "conditions": [
    {
      "id": "compare-two-values",
      "objectClass": "System",
      "sid": 990000000000132,
      "parameters": {
        "first-value": "Dilema_Tipo",
        "comparison": 0,
        "second-value": "\"Martírio\""
      }
    }
  ],
  "actions": [
    {
      "id": "set-eventvar-value",
      "objectClass": "System",
      "sid": 990000000000133,
      "parameters": {
        "variable": "Karma",
        "value": "Karma + 2"
      }
    },
    {
      "id": "set-eventvar-value",
      "objectClass": "System",
      "sid": 990000000000134,
      "parameters": {
        "variable": "Fe",
        "value": "max(0, Fe - 5)"
      }
    },
    {
      "id": "set-eventvar-value",
      "objectClass": "System",
      "sid": 990000000000135,
      "parameters": {
        "variable": "Martírio_Protegeu",
        "value": "1"
      }
    },
    {
      "id": "set-eventvar-value",
      "objectClass": "System",
      "sid": 990000000000136,
      "parameters": {
        "variable": "Martírio_Ocorreu",
        "value": "1"
      }
    }
  ]
}
```

*(Adicionar +15 Felicidade ao Adepto `Dilema_Dono_UID`.)*

- [ ] **Step 5: Validar e commitar**

```bash
python -c "import json; json.load(open('eventSheets/Folha de eventos 1.json')); print('OK')"
git add "eventSheets/Folha de eventos 1.json"
git commit -m "feat: adicionar dilema Martírio (único) com karma e flags"
```

---

## Task 10 (Fase 2): Final Secreto — A Encarnação

**Arquivo:** `eventSheets/Folha de eventos 1.json`

Adiciona verificação de todas as 5 flags antes de exibir os finais normais. Se todas ativas + Fe 45–70, exibe dilema especial da Encarnação com btn_Dilema_Sim e btn_Dilema_Nao.

- [ ] **Step 1: Adicionar bloco de verificação do final secreto ANTES dos outros finais**

No bloco de Era 3 da Task 7 (SID `990000000000060`), inserir o primeiro child block como verificador de final secreto:

```json
{
  "eventType": "block",
  "sid": 990000000000140,
  "conditions": [
    {
      "id": "compare-two-values",
      "objectClass": "System",
      "sid": 990000000000141,
      "parameters": { "first-value": "Justica_Perdoou", "comparison": 0, "second-value": "1" }
    },
    {
      "id": "compare-two-values",
      "objectClass": "System",
      "sid": 990000000000142,
      "parameters": { "first-value": "Voz_Acalmou", "comparison": 0, "second-value": "1" }
    },
    {
      "id": "compare-two-values",
      "objectClass": "System",
      "sid": 990000000000143,
      "parameters": { "first-value": "Herege_Permitiu", "comparison": 0, "second-value": "1" }
    },
    {
      "id": "compare-two-values",
      "objectClass": "System",
      "sid": 990000000000144,
      "parameters": { "first-value": "Martírio_Protegeu", "comparison": 0, "second-value": "1" }
    },
    {
      "id": "compare-two-values",
      "objectClass": "System",
      "sid": 990000000000145,
      "parameters": { "first-value": "Raio_Nunca_Adepto", "comparison": 0, "second-value": "1" }
    },
    {
      "id": "compare-two-values",
      "objectClass": "System",
      "sid": 990000000000146,
      "parameters": { "first-value": "Fe", "comparison": 5, "second-value": "45" }
    },
    {
      "id": "compare-two-values",
      "objectClass": "System",
      "sid": 990000000000147,
      "parameters": { "first-value": "Fe", "comparison": 4, "second-value": "70" }
    }
  ],
  "actions": [
    {
      "id": "set-visible",
      "objectClass": "btn_Dilema_Sim",
      "sid": 990000000000148,
      "parameters": { "visibility": "visible" }
    },
    {
      "id": "set-visible",
      "objectClass": "btn_Dilema_Nao",
      "sid": 990000000000149,
      "parameters": { "visibility": "visible" }
    },
    {
      "id": "set-text",
      "objectClass": "txt_Dilema_Titulo",
      "sid": 990000000000150,
      "parameters": { "text": "\"A ENCARNAÇÃO\"" }
    },
    {
      "id": "set-text",
      "objectClass": "txt_Dilema_Corpo",
      "sid": 990000000000151,
      "parameters": {
        "text": "\"Seu povo está inteiro.\" & newline & \"Mas sua presença os\" & newline & \"mantém presos à sua\" & newline & \"sombra. Você existe\" & newline & \"para eles, ou eles\" & newline & \"existem para você?\""
      }
    },
    {
      "id": "set-text",
      "objectClass": "txt_Dilema_Efeitos",
      "sid": 990000000000152,
      "parameters": { "text": "\"SIM DESCER\" & newline & \"NÃO PERMANECER\"" }
    },
    {
      "id": "set-eventvar-value",
      "objectClass": "System",
      "sid": 990000000000153,
      "parameters": { "variable": "Dilema_Tipo", "value": "\"Encarnacao\"" }
    }
  ]
}
```

*(Este bloco usa `"stop-or-else"` implicitamente — se suas condições forem verdadeiras, os blocos de final normais abaixo não devem executar. Em Construct 3, adicionar um else ou usar uma variável `Final_Definido` para evitar que os outros finais sobrescrevam o texto da Encarnação.)*

- [ ] **Step 2: Adicionar handler SIM da Encarnação (final secreto)**

No btn_Dilema_Sim, adicionar child block para Dilema_Tipo = "Encarnacao":

```json
{
  "eventType": "block",
  "sid": 990000000000155,
  "conditions": [
    {
      "id": "compare-two-values",
      "objectClass": "System",
      "sid": 990000000000156,
      "parameters": { "first-value": "Dilema_Tipo", "comparison": 0, "second-value": "\"Encarnacao\"" }
    }
  ],
  "actions": [
    {
      "id": "set-visible",
      "objectClass": "btn_Dilema_Sim",
      "sid": 990000000000157,
      "parameters": { "visibility": "invisible" }
    },
    {
      "id": "set-visible",
      "objectClass": "btn_Dilema_Nao",
      "sid": 990000000000158,
      "parameters": { "visibility": "invisible" }
    },
    {
      "id": "set-text",
      "objectClass": "txt_Dilema_Titulo",
      "sid": 990000000000159,
      "parameters": { "text": "\"A ENCARNAÇÃO\"" }
    },
    {
      "id": "set-text",
      "objectClass": "txt_Dilema_Corpo",
      "sid": 990000000000160,
      "parameters": {
        "text": "\"Você desceu sem anunciar.\" & newline & \"Viveu entre eles,\" & newline & \"morreu entre eles.\" & newline & \"Eles nunca souberam\" & newline & \"que foi você.\""
      }
    },
    {
      "id": "set-text",
      "objectClass": "txt_Dilema_Efeitos",
      "sid": 990000000000161,
      "parameters": { "text": "\"\""  }
    }
  ]
}
```

- [ ] **Step 3: Adicionar handler NÃO da Encarnação (cai para Equilíbrio)**

No btn_Dilema_Nao, adicionar child block para Dilema_Tipo = "Encarnacao":

```json
{
  "eventType": "block",
  "sid": 990000000000162,
  "conditions": [
    {
      "id": "compare-two-values",
      "objectClass": "System",
      "sid": 990000000000163,
      "parameters": { "first-value": "Dilema_Tipo", "comparison": 0, "second-value": "\"Encarnacao\"" }
    }
  ],
  "actions": [
    {
      "id": "set-visible",
      "objectClass": "btn_Dilema_Sim",
      "sid": 990000000000164,
      "parameters": { "visibility": "invisible" }
    },
    {
      "id": "set-visible",
      "objectClass": "btn_Dilema_Nao",
      "sid": 990000000000165,
      "parameters": { "visibility": "invisible" }
    },
    {
      "id": "set-text",
      "objectClass": "txt_Dilema_Titulo",
      "sid": 990000000000166,
      "parameters": { "text": "\"O PASTOR\"" }
    },
    {
      "id": "set-text",
      "objectClass": "txt_Dilema_Corpo",
      "sid": 990000000000167,
      "parameters": {
        "text": "\"Eles acordam. Fizeram\" & newline & \"filhos, enterraram os mortos.\" & newline & \"Ainda acendem a vela.\" & newline & \"Não perguntam mais\" & newline & \"por você.\""
      }
    }
  ]
}
```

- [ ] **Step 4: Garantir que os finais normais não sobrescrevam o final secreto**

Adicionar uma variável `Final_Definido` (SID `990000000000009`, initialValue `"0"`) ao event sheet. Setar para `"1"` ao início do bloco da Encarnação (Task 10 Step 1). Adicionar condição `Final_Definido = 0` a cada um dos blocos de final normais (Task 7 Steps 2).

- [ ] **Step 5: Validar e commitar**

```bash
python -c "import json; json.load(open('eventSheets/Folha de eventos 1.json')); print('OK')"
git add "eventSheets/Folha de eventos 1.json"
git commit -m "feat: adicionar final secreto A Encarnação com dilema especial"
```

**Teste manual:** Usar variáveis de debug para forçar todas as flags = 1 e Fe = 60, completar Era 3 e verificar que o dilema da Encarnação aparece antes dos finais normais.

---

## Apêndice: Ordem de Comparação em Construct 3

| Valor | Operador |
|-------|----------|
| 0 | = (igual) |
| 1 | ≠ (diferente) |
| 2 | < (menor que) |
| 3 | > (maior que) |
| 4 | ≤ (menor ou igual) |
| 5 | ≥ (maior ou igual) |

## Apêndice: SIDs Reservados Neste Plano

| Faixa | Uso |
|-------|-----|
| `990000000000001`–`990000000000009` | Variáveis globais |
| `990000000000010`–`990000000000029` | Zonas de fé (Task 3) |
| `990000000000030`–`990000000000039` | Karma dilemas existentes (Task 4) |
| `990000000000040`–`990000000000049` | Poder Bênção (Task 5) |
| `990000000000050`–`990000000000059` | Poder Silêncio (Task 6) |
| `990000000000060`–`990000000000099` | Sistema de finais (Task 7) |
| `990000000000100`–`990000000000119` | Dilema Herege (Task 8) |
| `990000000000120`–`990000000000139` | Dilema Martírio (Task 9) |
| `990000000000140`–`990000000000169` | Final Secreto (Task 10) |
