# CLAUDE.md

Contexto técnico de **PorriLand**, jogo 2D em Construct 3 para a game jam **Overpower**.

## Convenções

- Usar português brasileiro em objetos, grupos, variáveis e comentários.
- Estados oficiais: `"Ceticismo"`, `"Equilibrado"` e `"Fanatismo"`.
- Ignorar `*.uistate.json`.
- Preferir o editor/MCP do Construct 3; editar JSON manualmente apenas em mudanças pequenas.

## Núcleo Atual

- Adeptos, Céticos e Fanáticos.
- Movimento em grid e conversão de Adeptos por distância.
- Felicidade, procriação, dúvida e trauma.
- Fé global e evolução por equilíbrio.
- Poderes `Raio` e `Sussurro` com botões dinâmicos e cooldown.
- Menu lateral de objetivos.
- Dilemas raros que pausam o jogo e oferecem Sim/Não com benefício e custo.
- Ações de dilema que podem marcar um Adepto para uso de poder.

## Objetos Relevantes

```text
Porri_Adepto
Porri_Cetico
Porri_Fanatico
Porri´s
btn_Poderes
cd_overlay
Borda_Poder
Raio
Sussurro
Oratorio_Painel
Oratorio_Aba
sf_Objetivos
Filtro_Dilema
Painel_Dilema
sf_Dilema_Titulo
sf_Dilema_Corpo
sf_Dilema_Efeitos
btn_Dilema_Sim
btn_Dilema_Nao
MarcadorAlvoOracao
Controlador
```

`Oratorio_Painel`, `Oratorio_Aba`, `Oratorio_Aberto` e `MarcadorAlvoOracao` têm nomes legados. Hoje representam, respectivamente, painel de objetivos, aba de objetivos, abertura dos objetivos e mira de uma ação de dilema.

## Variáveis Globais

```text
Dilema_Acao_Pendente
Dilema_Alvo_UID
Dilema_Dono_UID
Dilema_Tipo
Dilema_Ativo
Alcance_Agarrao_Fanatico
Tempo_Agarrao_Matar
Fe
Era
Estado
Oracoes_Atendidas
Oracoes_Negadas
Oratorio_Aberto
Mortes
Fanaticos_Vivos
Ceticos_Vivos
Adeptos_Vivos
Tempo_Equilibrio
Poder_Selecionado
MaxPorris
```

## Regras Fixas

Conversão:

```text
Alcance_Conversao = 34
Chance = clamp(0.12 + ((50 - Porri_Adepto.Felicidade) / 500), 0.06, 0.22)
```

Fanáticos:

```text
Alcance_Agarrao_Fanatico = 44
Tempo_Agarrao_Matar = 3
```

Poderes:

- `Raio` mata ameaças e resolve `Raio_Adepto` no alvo marcado.
- `Sussurro` converte Céticos e resolve `Sussurro_Adepto` no alvo marcado.
- Clicar no poder já selecionado limpa `Poder_Selecionado` e remove `Borda_Poder`.

## Objetivos E Dilemas

O grupo oficial é:

```text
Objetivos e Dilemas
```

O painel lateral mostra objetivos da Era. O timer `"Dilema"` ocorre a cada 90–150 segundos se não houver modal nem ação pendente.

Ao abrir um dilema:

- `Dilema_Ativo = 1`.
- `time scale = 0`.
- O filtro escurece o jogo.
- O painel e os textos continuam opacos.
- `btn_Dilema_Sim` e `btn_Dilema_Nao` ficam visíveis.

Ao decidir, toda a UI é ocultada e movida para `-2000, -2000`, o jogo volta para `time scale = 1` e a consequência é aplicada.

Dilemas atuais:

```text
Festa_Esperanca
Justica_Vizinho
Voz_Acalma
```

`Justica_Vizinho` aceita `Raio` no Adepto marcado. Recusar converte `Dilema_Dono_UID` em Cético.

`Voz_Acalma` aceita `Sussurro` no Adepto marcado. Recusar aumenta Fé e reduz a Felicidade do alvo.

## Mecânica Antiga Removida

Não recriar fila, cards, expiração ou QTE por `BalaoOracao`. Estes objetos foram removidos:

```text
BalaoOracao
sf_Pedido
Txt_Oracao_Titulo
Txt_Oracao_Corpo
Txt_Oracao_Efeito
Barra_Oracao
btn_Atender
btn_Negar
MarcadorOracao
```

Os únicos botões de decisão são `btn_Dilema_Sim` e `btn_Dilema_Nao`.

## Validação

Não há suíte Python em `tests`. Usar `validate_project` do MCP, parse de todos os JSONs, revisão do `git diff` e reabertura no Construct 3.

Último estado: projeto válido, sem erros ou avisos do validador estrutural.
