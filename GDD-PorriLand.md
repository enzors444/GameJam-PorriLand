# Documento De Game Design — PorriLand

> **Tema da jam:** Overpower  
> **Engine:** Construct 3  
> **Plataforma:** PC, Web/Desktop  
> **Viewport:** 1280x720  
> **Estilo:** pixel art simples, leitura rápida, personagens por cor

## 1. Visão Geral

**PorriLand** é um god game minimalista em tempo real. O jogador guia uma civilização de pequenos seres chamados **Porris** da Tribo até a Metrópole.

O jogador possui poderes divinos absolutos, mas o objetivo não é maximizar a Fé. A civilização só evolui quando permanece viva e equilibrada entre dois extremos:

- **Ceticismo:** Fé baixa, Céticos se espalham e a população perde confiança.
- **Fanatismo:** Fé alta demais, Fanáticos surgem e causam caos.

O tema **Overpower** está no contraste entre poder e responsabilidade: toda solução divina cria outra tensão.

## 2. Pilares De Design

### O Peso Do Poder

O jogador resolve problemas rapidamente, mas cada intervenção afeta Fé, Felicidade, população ou mortes.

### Equilíbrio Dinâmico

O caminho saudável é manter a Fé entre 50 e 70. Os extremos alteram o comportamento dos Porris e impedem a evolução.

### Dilemas Sem Resposta Perfeita

Orações são raras e importantes. Elas pausam o jogo e exigem Sim ou Não. Nenhuma decisão é totalmente boa: toda opção combina benefício e custo.

### Ação Divina Concreta

Algumas escolhas não terminam no botão. Aceitar pode marcar um Porri no mapa e exigir o poder correto no alvo correto.

## 3. População

### Adeptos

- São os fiéis comuns e contam como vida.
- Geram Fé passivamente.
- Possuem Felicidade de 0 a 100.
- Felicidade alta permite procriação.
- Felicidade baixa pode causar conversão espontânea em Cético.
- Se o número de Adeptos chegar a zero, a civilização perde sua base vital.

### Céticos

- Reduzem Fé.
- Caminham aleatoriamente ou seguem o Adepto mais próximo.
- Reduzem a Felicidade de Adeptos próximos pela aura de dúvida.
- Convertem por distância, com chance influenciada pela Felicidade do alvo.

### Fanáticos

- Surgem quando a Fé está alta demais.
- Não contam como vida.
- Priorizam Céticos; sem Céticos, podem perseguir Adeptos.
- Agarram o alvo e matam depois de 3 segundos dentro do alcance.
- Espalham trauma e queda de Felicidade.

## 4. Fé E Estados

| Fé | Estado | Efeito principal |
|---:|---|---|
| 0–49 | Ceticismo | Céticos pressionam mais e a sociedade perde confiança. |
| 50–70 | Equilibrado | Único estado que acumula progresso para evoluir. |
| 71–100 | Fanatismo | Fanáticos surgem e aumentam o caos. |

Fórmula atual:

```text
Fe += Porri_Adepto.Count * 0.02
Fe -= Porri_Cetico.Count * 0.08
Fe += Porri_Fanatico.Count * 0.04
Fe = clamp(Fe, 0, 100)
```

## 5. Felicidade

| Faixa | Estado | Resultado |
|---:|---|---|
| 75–100 | Feliz | Pode procriar. |
| 31–74 | Estável | Não procria nem cai imediatamente. |
| 0–30 | Infeliz | Pode virar Cético. |

Raio, Fanáticos e dilemas podem gerar trauma. A Felicidade se recupera quando o Adepto passa tempo sem eventos traumáticos.

## 6. Movimento E Conversão

Porris usam movimento em grid. Adeptos escolhem direções aleatórias; Céticos também podem usar `"Seguir"`.

Conversão:

```text
distance(Cetico, Adepto) <= 34
chance = clamp(0.12 + ((50 - Felicidade) / 500), 0.06, 0.22)
```

A conversão não usa sobreposição como regra principal.

## 7. Poderes Divinos

| Poder | Cooldown | Uso normal | Uso em dilema |
|---|---:|---|---|
| Raio | 4s | Mata Céticos e Fanáticos; traumatiza. | Executa o alvo de `Justica_Vizinho`. |
| Sussurro | 7s | Converte Céticos silenciosamente. | Acalma o alvo de `Voz_Acalma`. |

Os botões aparecem no canto inferior direito, têm 64×64 e são criados automaticamente conforme `eraDesbloqueio`. Novos botões crescem para a esquerda.

O jogador pode selecionar ou desmarcar um poder. Uma borda amarela indica a seleção; o overlay indica cooldown.

## 8. Progressão E Objetivos

Fluxo planejado:

```text
Tribo -> Vila -> Cidade -> Metrópole
```

O antigo painel lateral de orações agora é o **menu de objetivos**. Ele mostra requisitos e marca os já concluídos.

Objetivos da Era 1:

- 30 Adeptos.
- Fé entre 50 e 70.
- Permanecer 5 segundos em `"Equilibrado"`.

O botão de evolução só deve avançar quando todos forem cumpridos.

## 9. Dilemas De Oração

Orações não são mais uma fila de pedidos pequenos com expiração. São eventos raros e de alto impacto.

Regras:

- Intervalo atual: 90–150 segundos.
- Apenas um dilema ou uma ação pendente por vez.
- O jogo pausa completamente durante a decisão.
- Um filtro preto cobre o fundo; o painel permanece opaco.
- O jogador escolhe Sim ou Não.
- Ambas as opções sempre trazem ganho e perda.
- A UI desaparece imediatamente após a decisão.
- Se Sim exigir um poder, o jogo volta e uma mira marca o alvo.
- O próximo dilema só é agendado após a ação pendente terminar.

### Preço Da Esperança

O povo pede uma festa para esquecer o medo.

| Escolha | Benefício | Custo |
|---|---|---|
| Sim | +15 Felicidade geral | -12 Fé e 3 Adeptos viram Céticos |
| Não | +10 Fé | -8 Felicidade geral e 1 Adepto vira Fanático |

### Justiça Divina

Um fiel pede punição porque o vizinho deu em cima de sua mulher.

| Escolha | Benefício | Custo |
|---|---|---|
| Sim | +10 Fé ao cumprir | Um Adepto marcado deve morrer com Raio |
| Não | +10 Fé | Quem orou vira Cético |

Ao aceitar, a mira marca um Adepto aleatório diferente do orador. O jogador deve selecionar `Raio` e clicar nesse alvo.

### Uma Voz Que Acalma

Um Adepto não consegue dormir e pede a voz divina.

| Escolha | Benefício | Custo |
|---|---|---|
| Sim | +20 Felicidade no alvo | -8 Fé e exige Sussurro no alvo marcado |
| Não | +8 Fé | -15 Felicidade no alvo |

## 10. Interface

- Topo/esquerda: contadores e estado.
- Barra de Fé: mostra posição entre Ceticismo, Equilíbrio e Fanatismo.
- Lateral direita: menu suspenso de objetivos.
- Canto inferior direito: poderes e cooldowns.
- Centro: modal opaco de dilema quando uma oração rara acontece.
- Mapa: mira acima do alvo quando existe ação aceita.

Os textos do menu e do modal usam Sprite Font do projeto. Espaços devem permanecer espaços; não usar hífens como substituto.

## 11. Mecânica Antiga Removida

Não fazem mais parte do design:

- Fila de orações.
- Cards simultâneos.
- Barra de tempo por pedido.
- Pedido ignorado por expiração.
- `btn_Atender` e `btn_Negar` em cada card.
- QTE ligada a `BalaoOracao`.
- Orações secretas e ganchos de finais secretos antigos.

Algumas ideias foram recicladas como dilemas raros, mas a implementação antiga não deve voltar.

## 12. MVP Atual

- [x] Movimento dos Porris
- [x] Céticos com perseguição e conversão por distância
- [x] Felicidade, dúvida e procriação
- [x] Fanáticos
- [x] Fé global e estados
- [x] UI e cursor customizado
- [x] Raio e Sussurro
- [x] Botões dinâmicos, seleção e cooldown
- [x] Menu lateral de objetivos
- [x] Modal de dilemas com pausa
- [x] Dilemas com consequências positivas e negativas
- [x] Alvos marcados para Raio e Sussurro
- [ ] Desastres
- [ ] Áudio
- [ ] Eras 2–4 completas

## 13. Próximas Etapas

1. Balancear frequência e valores dos dilemas.
2. Adicionar novos dilemas por Era.
3. Implementar desastres.
4. Completar evolução visual e mecânica das Eras.
5. Polir áudio e feedback.
