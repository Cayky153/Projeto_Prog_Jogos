# Entidades do jogo — AutoRunner

**Integrantes:** Antônio Pedro Cayky Do Nascimento Pereira

**Checkpoint 2 · Programação para Jogos I · Entrega: sexta, 02/10/2026**

## Contextualização
Inspirado no jogo mobile "Sonic Runners", o jogo será de plataforma 2D do tipo autorunner, ou seja, o personagem do jogador está sempre em movimento, com o jogador não podendo controlar isso, com seu único input sendo para pular para desviar de obstáculos. Para simular esse movimento, eu optei por não fazer com que o jogador se mexesse, e sim todo o resto se mexesse em direção ao jogador, com plataformas, obstáculos e inimigos tendo velocidade no eixo X, mas o jogador não tendo.

## 1. Entidades

O que se move, muda de estado ou reage a algo? Uma por linha.

- Jogador
- Inimigo
- Plataforma
- Obstáculo
- Chão
- FundoParallax
- GameManager

## 2. Propriedades

O que cada entidade sabe sobre si?

| Entidade | Propriedades |
|---|---|
| Jogador | posicaoX, posicaoY, width, height, isAlive, podePular, gravidade, velocidadeY, alturaLimite |
| Inimigo | posicaoX, posicaoY, velocidade, width, height, isAlive |
| Plataforma | posicaoX, posicaoY, velocidade, width, height |
| Obstáculo | posicaoX, posicaoY, velocidade, width, height |
| Chão | posicaoX, posicaoY, width, height, velocidade |
| FundoParallax | imagem, posicaoX, posicaoY, velocidade, width |
| GameManager | velocidadeMundo, velocidadeBase, velocidadeMaxima, pontuacao, distanciaPercorrida, inimigos, plataformas, chao, obstaculos |

## 3. Comportamentos

O que cada entidade faz a cada quadro?

| Entidade | Comportamentos |
|---|---|
| Jogador | lê a entrada (pulo), aplica gravidade, atualiza posicaoY, checa se está no chão, checa se caiu abaixo da altura limite (se sim, morte), atualiza o estado de isAlive, atualiza animação, desenha |
| Inimigo | move no eixo X, checa se está vivo |
| Plataforma | move no eixo X |
| Obstáculo | move no eixo X |
| Chão | move no eixo X |
| FundoParallax | move no eixo X |
| GameManager | spawna as entidades exceto o jogador, controla a velocidade delas também, testa as colisões |

## 4. Colisões

O que colide com o quê? A reação é igual para todo par?

| Quem | Com quem | O que acontece |
|---|---|---|
| Jogador | Inimigo | Irá depender da colisão, se a colisão for horizontal ou vertical abaixo do inimigo, o jogador morre, com o isAlive do jogador recebendo false, se a colisão for vertical com o jogador acima do inimigo, o isAlive do inimigo recebe false, desaparecendo da fase, com a velocidadeY de jogador aumentando para simular um impulso, e ele recuperando seu pulo, com podePular saindo de false para true |
| Jogador | Obstáculo | Jogador morre, com seu isAlive recebendo false |
| Jogador | Plataforma | Se a colisão for acima da plataforma, a velocidadeY do jogador cessa, mantendo ele no "chão", se a colisão for embaixo, a velocidadeY do jogador cessa, fazendo com que ele volte a cair, se a colisão for na esquerda da plataforma, a plataforma e o cenário "param" para simular o jogador parando |
| Jogador | Chão | A velocidadeY do jogador cessa, mantendo ele no "chão", se a colisão for na esquerda do chão, o chão e o cenário "param" para simular o jogador parando |

A reação é a mesma para todos os pares? Se não, onde ela muda:

Não. A reação muda de dois jeitos:

- **Entre pares diferentes:** Jogador × Obstáculo sempre mata o jogador, enquanto Jogador × Inimigo pode matar o jogador ou o inimigo, e Jogador × Plataforma/Chão só interrompe o movimento.
- **Dentro do mesmo par, dependendo do lado da colisão:** em Jogador × Inimigo, colidir por cima mata o inimigo e dá impulso, e colidir pelos lados ou por baixo mata o jogador. Em Jogador × Plataforma, colidir por cima mantém o jogador no chão, por baixo zera a velocidadeY e ele volta a cair, e pela esquerda o cenário para.

Os pares Jogador × Plataforma e Jogador × Chão têm reações quase idênticas, então podem compartilhar o mesmo código.

## 5. Comunicação

Uma entidade aciona ou lê a outra diretamente, ou por um terceiro (o jogo, o mundo, um gerenciador)?

- GameManager → Inimigo / Plataforma / Obstáculo / Chão / FundoParallax: **direto**. O GameManager cria (spawna) cada uma e altera a propriedade velocidade delas. Ele é o único responsável por aumentar ou reduzir a velocidade do cenário, incluindo zerá-la quando o jogador bate na lateral de uma Plataforma ou Chão (e restaurá-la quando o jogador deixa de estar bloqueado).
- Jogador ↔ Inimigo / Obstáculo / Plataforma / Chão: **por um terceiro (o GameManager)**. A cada quadro, o GameManager testa as colisões e aplica a reação: muda isAlive do Jogador e do Inimigo, altera velocidadeY e podePular do Jogador, ou zera a velocidade do cenário. O Jogador e as outras entidades não se conhecem.
- GameManager → Placar: **direto**. O GameManager entrega pontuacao e distanciaPercorrida para o Placar desenhar.

## 6. Falsas entidades

Algo parece entidade, mas não se atualiza sozinho (placar, cenário, som, câmera)?

- Placar
- Câmera

## 7. Repetições

Duas ou mais entidades repetem o mesmo comportamento? Qual, e em quais?

- **Mover no eixo X para a esquerda:** Inimigo, Plataforma, Obstáculo, Chão e FundoParallax.
- **Reação de colisão por lado:** Jogador × Plataforma e Jogador × Chão repetem a mesma lógica (parar por cima, cair por baixo, parar o cenário pela esquerda).

## Pergunta que mais travou
A 5, pois antes não havia feito um GameManager, então ficou algumas inconsistências como "Quem vai atualizar o placar?", "Como vai as velocidades vao ser atualizadas? vai ser uma por uma?" "Quem vai criar os objetos do cenário" então fiquei nessa dúvida por muito tempo, o que dificultou o 5, até que eu tive a ideia de criar um gamemanager para unificar isso