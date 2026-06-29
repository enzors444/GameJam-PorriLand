# AGENTS.md

Este arquivo orienta o Codex ao trabalhar neste repositório.

## O Que É Este Projeto

**PorriLand** é um jogo 2D feito no **Construct 3** para uma game jam com o tema **Overpower**.

É um god game minimalista em tempo real. O jogador guia uma civilização de pequenos personagens chamados **Porris**, mantendo a população viva e a Fé equilibrada entre **Ceticismo** e **Fanatismo**.

A linguagem do projeto é **português brasileiro**. Manter nomes de objetos, grupos, variáveis e comentários em português.

## Formato Do Projeto

O projeto está salvo no formato de pasta do Construct 3.

- `project.c3proj`: manifesto principal.
- `objectTypes`: tipos de objetos.
- `families`: famílias.
- `layouts`: layouts/cenas.
- `eventSheets`: lógica visual.
- `images`: sprites.
- `*.uistate.json`: estado visual do editor; ignorar.

Preferir o MCP/editor visual do Construct 3. Só editar JSON manualmente em mudanças pequenas, precisas e seguras.

## Estado Atual

Fechado/funcionando:

- 10 Adeptos iniciais.
- Movimento em grid.
- Céticos com modo `"Seguir"` e conversão por distância.
- Felicidade individual, dúvida, procriação e queda para Ceticismo.
- Fanáticos com perseguição, agarrão e morte por tempo.
- UI com contadores, estado e barra de Fé.
- Cursor customizado.
- Poderes da Era 1: `Raio` e `Sussurro`.
- Botões de poder criados automaticamente conforme a Era.
- Seleção alternável, borda de seleção e cooldown visual.
- Menu lateral de objetivos.
- Dilemas raros de oração com pausa total e escolhas Sim/Não.
- Ações aceitas que podem exigir `Raio` ou `Sussurro` em um alvo marcado.

Ainda planejado:

- Desastres.
- Áudio.
- Balanceamento fino.
- Eras além dos ganchos atuais.

## Modelo Central

- `Porri_Adepto`: cidadão fiel; conta como vida.
- `Porri_Cetico`: ameaça de Ceticismo.
- `Porri_Fanatico`: ameaça de Fanatismo.
- Família principal: `Porri´s`.

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

Estados oficiais:

```text
"Ceticismo"
"Equilibrado"
"Fanatismo"
```

Não usar variações.

## Variáveis De Instância

### Família `Porri´s`

```text
Direcao
Ultimo_X
Ultimo_Y
Altura_Padrão
Largura_Padrão
```

`Direcao` pode ser `"Cima"`, `"Baixo"`, `"Esquerda"`, `"Direita"`, `"Parado"` ou `"Seguir"`.

### `Porri_Adepto`

```text
Felicidade
Tempo_Sem_Trauma
Imune
```

### `Porri_Fanatico`

```text
modelo
Tempo_Agarrao
Alvo_UID
```

### `btn_Poderes`

```text
idPoder
eraDesbloqueio
cd
cdMax
```

## Grupos De Eventos

```text
Setup
Porri
    Movimento e IA
        Limites do Mapa
        Separar Porris
        Animação Procedural
    Felicidade
        Procriar
    Céticos
        Nascimento
        Dúvida
    Fanáticos
UI e Contadores
    Mouse
    Atualizar Contadores
    Atualizar Fé
Evolução
Debug
    Debug Felicidade
    Debug Ceticos
Poderes
    Poderes - Seleção
    Poderes - Uso
    Poderes - Cooldown
    Poderes - Feedback Visual
Objetivos e Dilemas
```

## Regras Importantes

### Conversão

Não usar sobreposição como regra principal.

```text
distance(Porri_Cetico.X, Porri_Cetico.Y, Porri_Adepto.X, Porri_Adepto.Y) <= 34
Chance = clamp(0.12 + ((50 - Porri_Adepto.Felicidade) / 500), 0.06, 0.22)
```

### Fanáticos

- Surgem em `"Fanatismo"`.
- `Alcance_Agarrao_Fanatico = 44`.
- `Tempo_Agarrao_Matar = 3`.
- Priorizam Céticos; sem Céticos, podem mirar Adeptos.
- Resetam alvo e agarrão se o alvo some ou sai do alcance.

### Fé

```text
Fe += Porri_Adepto.Count * 0.02
Fe -= Porri_Cetico.Count * 0.08
Fe += Porri_Fanatico.Count * 0.04
Fe = clamp(Fe, 0, 100)
```

### Poderes

- `Raio`: mata Céticos/Fanáticos e resolve o dilema `Justica_Vizinho` no alvo marcado.
- `Sussurro`: converte Céticos e resolve o dilema `Voz_Acalma` no Adepto marcado.
- `Poder_Selecionado` vazio significa nenhum poder selecionado.
- Clicar novamente no poder selecionado deve desmarcá-lo e destruir `Borda_Poder`.
- Os botões são criados por dados de animação, ID, Era e cooldown; não colocar botões manualmente no layout.

## Objetivos

O painel lateral antigo foi reaproveitado como menu de objetivos.

- `Oratorio_Painel` e `Oratorio_Aba` são nomes legados, mas hoje pertencem aos objetivos.
- `Oratorio_Aberto` controla a abertura.
- `sf_Objetivos` acompanha o painel.
- Na Era 1: 30 Adeptos, Fé entre 50 e 70 e 5 segundos em equilíbrio.

## Dilemas De Oração

O sistema antigo de cartões temporizados foi removido. Não reintroduzir:

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
fila de orações
timer individual de pedido
QTE antiga por BalaoOracao
```

O sistema atual usa somente o modal de dilema:

```text
Filtro_Dilema
Painel_Dilema
sf_Dilema_Titulo
sf_Dilema_Corpo
sf_Dilema_Efeitos
btn_Dilema_Sim
btn_Dilema_Nao
MarcadorAlvoOracao
```

`MarcadorAlvoOracao` mantém o nome legado, mas agora é apenas a mira da ação aceita e não possui `oracaoUID`.

Fluxo:

1. O timer raro `"Dilema"` dispara entre 90 e 150 segundos.
2. `Dilema_Ativo = 1` e o jogo pausa com `time scale = 0`.
3. O modal é opaco; apenas `Filtro_Dilema` escurece o fundo.
4. Sim e Não sempre possuem benefício e custo.
5. A escolha fecha e move a UI para fora da tela, depois restaura `time scale = 1`.
6. Se houver ação pendente, o próximo dilema só é agendado quando a ação for concluída ou o alvo deixar de existir.

Dilemas atuais:

- `Festa_Esperanca`: consequências imediatas para Felicidade, Fé e composição da população.
- `Justica_Vizinho`: Sim marca um Adepto para `Raio`; Não transforma quem orou em Cético e concede Fé.
- `Voz_Acalma`: Sim exige `Sussurro` no Adepto marcado; Não concede Fé e reduz a Felicidade do alvo.

## Pontos De Atenção

- `Controlador` precisa existir no layout.
- Não editar `*.uistate.json`.
- Não reintroduzir santuário herege agora.
- Não renomear `Estado` nem suas strings.
- Usar `1`/`0` em expressões do Construct quando não for um booleano de propriedade.
- Manter os textos dos Sprite Fonts em caracteres suportados; evitar acentos nos textos exibidos se o atlas não os contiver.
- Não usar hífen no lugar de espaço.
- Não remover `btn_Dilema_Sim`/`btn_Dilema_Nao`; eles substituem os antigos `btn_Atender`/`btn_Negar`.

## Validação

Não há diretório `tests` neste estado do projeto. Validar com:

```text
MCP Construct 3: validate_project
JSON.parse nos arquivos .json e project.c3proj
reabrir o projeto no Construct 3
```

Última validação documentada: projeto válido, 0 erros e 0 avisos.

## Construct3 MCP Local

- Servidor em `.agents/mcp/construct3-mcp`.
- Configuração em `.codex/config.toml`.
- Começar por leitura e análise.
- Nunca usar `force` em exclusões sem autorização explícita.
- Confirmar IDs ACE e parâmetros em eventos existentes.
- Preservar grupos, ordem, seleção de objetos, `Else`, esperas e subeventos.
- Depois de mutações, validar JSON, revisar `git diff` e reabrir o projeto.
- Reiniciar o MCP após alterações diretas feitas no editor, pois o leitor mantém cache.
