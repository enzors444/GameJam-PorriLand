# Guia De Implementação No Construct 3 — PorriLand

Este guia descreve o estado técnico atual do projeto.

## 1. Convenções

- Usar português brasileiro.
- Estados oficiais: `"Ceticismo"`, `"Equilibrado"`, `"Fanatismo"`.
- Não editar `*.uistate.json`.
- Usar `1`/`0` em expressões numéricas do Construct.
- Manter os textos visuais dentro do conjunto de caracteres da Sprite Font.

## 2. Objetos Principais

### Porris

| Objeto | Função |
|---|---|
| `Porri_Adepto` | Cidadão fiel e vida da população. |
| `Porri_Cetico` | Persegue, influencia e converte Adeptos. |
| `Porri_Fanatico` | Persegue, agarra e mata. |
| `Porri´s` | Família compartilhada. |

### Poderes

| Objeto | Função |
|---|---|
| `btn_Poderes` | Botão dinâmico com ID, Era e cooldown. |
| `cd_overlay` | Overlay visual do cooldown. |
| `Borda_Poder` | Borda amarela do poder selecionado. |
| `Raio` | VFX e ação do Raio. |
| `Sussurro` | VFX e ação do Sussurro. |

### Objetivos

| Objeto | Função atual |
|---|---|
| `Oratorio_Painel` | Painel lateral de objetivos; nome legado. |
| `Oratorio_Aba` | Aba de abrir/fechar objetivos; nome legado. |
| `sf_Objetivos` | Texto dos requisitos da Era. |
| `btn_Evoluir` | Evolui quando os requisitos são cumpridos. |

### Dilemas

| Objeto | Função |
|---|---|
| `Filtro_Dilema` | Escurece somente o fundo. |
| `Painel_Dilema` | Moldura opaca do evento. |
| `sf_Dilema_Titulo` | Título em Sprite Font. |
| `sf_Dilema_Corpo` | Texto do problema. |
| `sf_Dilema_Efeitos` | Consequências de Sim e Não. |
| `btn_Dilema_Sim` | Aceita a oração. |
| `btn_Dilema_Nao` | Recusa a oração. |
| `MarcadorAlvoOracao` | Mira da ação pendente; nome legado. |

## 3. Variáveis Globais

```text
Dilema_Acao_Pendente = ""
Dilema_Alvo_UID = -1
Dilema_Dono_UID = -1
Dilema_Tipo = ""
Dilema_Ativo = 0

Fe = 60
Era = 1
Estado = "Equilibrado"
Poder_Selecionado = ""
MaxPorris = 200

Oracoes_Atendidas = 0
Oracoes_Negadas = 0
Oratorio_Aberto = 0

Adeptos_Vivos = 0
Ceticos_Vivos = 0
Fanaticos_Vivos = 0
Mortes = 0
Tempo_Equilibrio = 0

Alcance_Agarrao_Fanatico = 44
Tempo_Agarrao_Matar = 3
```

## 4. Grupos

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

## 5. Setup

No início do layout:

```text
Controlador -> iniciar timer "Dilema" por random(90,150)
Controlador -> iniciar timer "Desastre"
Controlador -> iniciar timer "Cetico_Aleatorio"
Ocultar cursor padrão
Criar 10 Porri_Adepto
Felicidade inicial = 70
Iniciar "Andar"
```

## 6. Movimento E Conversão

Adeptos escolhem direção aleatória. Céticos também podem escolher `"Seguir"` e mirar o Adepto mais próximo.

Conversão:

```text
A cada 0.5 segundos
Para cada Porri_Cetico
Escolher Adepto mais próximo
distance(Cetico, Adepto) <= 34
random(0,1) < clamp(0.12 + ((50 - Felicidade) / 500), 0.06, 0.22)
Imune não está setado
    criar Cético na posição
    iniciar "Andar"
    destruir Adepto
```

## 7. Fanáticos

Nascimento em `"Fanatismo"`:

```text
A cada 6 segundos
random(0,1) < 0.35
Porri_Adepto.Count > 0
Porri_Fanatico.Count < min(12, ceil(Porri_Adepto.Count / 3))
```

Agarrão:

```text
Alcance_Agarrao_Fanatico = 44
Tempo_Agarrao_Matar = 3
```

Priorizar Céticos. Sem Céticos, permitir Adeptos. Resetar `Alvo_UID` e `Tempo_Agarrao` quando o alvo ficar inválido.

## 8. Poderes Dinâmicos

Dados atuais:

```text
Animações: Relampago|Sussurro
IDs:       Raio|Sussurro
Eras:      1|1
Cooldowns: 4|7
```

No início:

1. Contar quantos poderes têm `eraDesbloqueio <= Era`.
2. Repetir a tabela.
3. Criar cada `btn_Poderes` liberado na layer `"UI"`.
4. Usar 64×64, Y 650 e X final 1200.
5. Posicionar por `1200 - (QtdLiberados - 1 - slot) * 72`.
6. Criar um `cd_overlay` associado por `Dono_UID`.

Seleção:

```text
Poder_Selecionado =
    Poder_Selecionado = btn_Poderes.idPoder
    ? ""
    : btn_Poderes.idPoder
```

Sempre destruir a borda anterior. Criar `Borda_Poder` somente se `Poder_Selecionado != ""`.

## 9. Menu De Objetivos

Clique em `Oratorio_Aba` quando não houver dilema:

```text
Oratorio_Aberto = 1 - Oratorio_Aberto
```

Movimento:

```text
Fechado: Oratorio_Painel.X -> 1280
Aberto:  Oratorio_Painel.X -> 870
Oratorio_Aba.X = Oratorio_Painel.X - 24
sf_Objetivos.X = Oratorio_Painel.X + 24
```

Era 1:

```text
Porri_Adepto.Count >= 30
Fe >= 50 e Fe <= 70
Tempo_Equilibrio >= 5
```

## 10. Gerar Um Dilema

Condições:

```text
Controlador -> ao terminar "Dilema"
Dilema_Ativo = 0
Dilema_Acao_Pendente = ""
Porri_Adepto.Count > 0
```

Ações principais:

```text
Dilema_Ativo = 1
Dilema_Tipo = choose(...)
Dilema_Dono_UID = UID de um Adepto aleatório
Dilema_Alvo_UID = UID de outro Adepto, quando necessário
Oratorio_Aberto = 0
time scale = 0
mostrar modal
mover modal para o topo
```

Se houver menos de dois Adeptos, usar `Festa_Esperanca` como fallback.

Tipos:

```text
Festa_Esperanca
Justica_Vizinho
Voz_Acalma
```

## 11. Fechar O Modal

Tanto Sim quanto Não devem:

```text
Dilema_Ativo = 0
ocultar Filtro_Dilema
ocultar Painel_Dilema
ocultar os três textos
ocultar btn_Dilema_Sim e btn_Dilema_Nao
mover todos para -2000, -2000
time scale = 1
```

Ocultar e mover é intencional: evita que a UI permaneça visível ou interativa após a escolha.

## 12. Consequências

### `Festa_Esperanca`

Sim:

```text
+15 Felicidade geral
-12 Fé
3 Adeptos aleatórios viram Céticos
```

Não:

```text
+10 Fé
-8 Felicidade geral
1 Adepto vira Fanático
```

O próximo timer começa imediatamente.

### `Justica_Vizinho`

Sim:

```text
Dilema_Acao_Pendente = "Raio_Adepto"
criar MarcadorAlvoOracao
```

Clique válido:

```text
Porri_Adepto.UID = Dilema_Alvo_UID
Poder_Selecionado = "Raio"
cooldown <= 0
    criar VFX
    Mortes += 1
    Fé += 10
    destruir alvo
    iniciar cooldown
    limpar ação e marcador
    agendar próximo dilema
```

Não:

```text
selecionar Porri_Adepto.UID = Dilema_Dono_UID
criar Cético na posição
destruir quem orou
Fé += 10
agendar próximo dilema
```

### `Voz_Acalma`

Sim:

```text
Dilema_Acao_Pendente = "Sussurro_Adepto"
usar Sussurro no Dilema_Alvo_UID
+20 Felicidade no alvo
-8 Fé
```

Não:

```text
+8 Fé
-15 Felicidade no alvo
```

## 13. Marcador E Fallback

Enquanto houver ação pendente:

```text
MarcadorAlvoOracao.X = alvo.X
MarcadorAlvoOracao.Y = alvo.Y - 34
```

Se o UID do alvo deixar de existir, destruir o marcador, limpar as variáveis e agendar o próximo dilema. Isso impede travamento do sistema.

## 14. Mecânica Removida

Não usar nem recriar:

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
MaxOracoesAtivas
Oracoes_Ignoradas
QTE por BalaoOracao
orações secretas antigas
```

## 15. Validação

Não existe atualmente `tests/verify_project.py`.

Validar com:

```text
Construct MCP -> validate_project
parse de project.c3proj e todos os JSONs
busca por referências dos objetos removidos
git diff --check
reabrir no Construct 3
```

Último estado documentado: 0 erros e 0 avisos estruturais.
