# Teste 4 — VGG16 com Data Augmentation Pesado

> Experimento da série de testes com VGG16 para classificação de ritmos cardíacos a partir de sinais de ECG representados como imagens, usando a estratégia de augmentation `D_pesado`.

## Sumário

1. [Objetivo](#objetivo)
2. [Configuração do experimento](#configuração-do-experimento)
3. [Resultados da melhor época](#resultados-da-melhor-época)
4. [Histórico de treinamento](#histórico-de-treinamento)
5. [Análise](#análise)
6. [Comparação com o Teste 1](#comparação-com-o-teste-1)
7. [Limitações](#limitações)
8. [Próximos passos](#próximos-passos)
9. [Conclusão](#conclusão)

## Objetivo

Avaliar o desempenho da arquitetura **VGG16** na classificação de sinais de ECG aplicando **data augmentation pesado**, com o objetivo de aumentar a variedade dos exemplos vistos no treinamento e, em tese, melhorar a generalização. O resultado é comparado ao baseline sem augmentation (Teste 1).

## Configuração do experimento

| Parâmetro | Valor |
|---|---|
| Arquitetura | VGG16 |
| Estratégia de augmentation | `D_pesado` — augmentation pesado |
| Número de classes | 6 |
| Épocas executadas | 14 |
| Melhor época | 4 |
| Tempo total de execução | 63,58 min (≈ 4,5 min/época) |

**Classes:** `AF`, `AFIB`, `SA`, `SB`, `SR`, `ST`

### Transformações do augmentation `D_pesado`

| Transformação | Parâmetros |
|---|---|
| `Resize` | 224 × 224 |
| `RandomAffine` | `degrees=3`, `translate=(0.06, 0.06)`, `scale=(0.90, 1.10)`, `fill=(255, 255, 255)` (rotação de ±3°, translação de até 6% e escala de ±10%; áreas novas em branco) |
| `ColorJitter` | `brightness=0.3`, `contrast=0.3` |
| `GaussianBlur` (via `RandomApply`) | `kernel_size=3`, `sigma=(0.1, 1.0)`, `p=0.3` |
| `ToTensor` | — |
| `RandomErasing` | `p=0.3`, `scale=(0.01, 0.04)`, `ratio=(0.3, 3.3)`, `value=1.0` (apaga pequenos trechos com branco, simulando ruído/artefatos do traçado) |
| `Normalize` | média `[0.485, 0.456, 0.406]`, desvio `[0.229, 0.224, 0.225]` |

**Regras adotadas para imagens de ECG (padronizadas na equipe):** sem *flip* horizontal ou vertical (inverteria o tempo e a polaridade das ondas); sem rotações grandes ou recortes agressivos (alterariam derivações e intervalos RR); áreas novas preenchidas com branco; augmentation aplicado **somente no treino**. Validação e teste usam apenas `Resize` + `ToTensor` + `Normalize`.

### Informações a completar

| Item | Valor |
|---|---|
| Dataset | _a preencher_ |
| Divisão treino/validação/teste | _a preencher_ |
| Pesos iniciais (ImageNet ou aleatórios) | _a preencher_ |
| Otimizador e *learning rate* | _a preencher_ |
| Batch size | _a preencher_ |
| Critério de parada / seleção do melhor modelo | _a preencher_ |
| Seed | _a preencher_ |
| GPU / ambiente | _a preencher_ |

## Resultados da melhor época

Melhor época: **4** (critério: menor `val_loss`, que coincide com a maior `val_acc`).

| Métrica | Treino | Validação |
|---|---:|---:|
| Acurácia | 60,40% | **71,99%** |
| Loss | 1,0927 | **0,8382** |

## Histórico de treinamento

| Época | Loss treino | Loss val. | Acc. treino | Acc. val. |
|---:|---:|---:|---:|---:|
| 1 | 1,1283 | 0,8586 | 58,38% | 70,71% |
| 2 | 1,1032 | 0,8753 | 59,59% | 66,71% |
| 3 | 1,0972 | 0,9053 | 60,16% | 68,82% |
| **4** | 1,0927 | **0,8382** | 60,40% | **71,99%** |
| 5 | 1,1032 | 0,8731 | 60,33% | 68,38% |
| 6 | 1,1016 | 0,9074 | 60,20% | 67,11% |
| 7 | 1,0924 | 0,9109 | 60,38% | 66,07% |
| 8 | 1,0956 | 0,8563 | 60,31% | 70,38% |
| 9 | 1,0997 | 0,8606 | 60,16% | 70,87% |
| 10 | 1,1036 | 0,8413 | 60,11% | 71,69% |
| 11 | 1,1049 | 0,8808 | 60,29% | 65,93% |
| 12 | 1,1029 | 0,8675 | 60,30% | 68,04% |
| 13 | 1,0976 | 0,8589 | 60,42% | 70,46% |
| 14 | 1,0953 | 0,8506 | 60,24% | 69,12% |

### Resumo estatístico (14 épocas)

| Métrica | Mínimo | Máximo | Média |
|---|---:|---:|---:|
| Acurácia de validação | 65,93% (ép. 11) | 71,99% (ép. 4) | 69,02% |
| Acurácia de treino | 58,38% (ép. 1) | 60,42% (ép. 13) | ≈ 60,0% |
| Loss de validação | 0,8382 (ép. 4) | 0,9109 (ép. 7) | — |
| Loss de treino | 1,0924 (ép. 7) | 1,1283 (ép. 1) | — |

## Análise

**Pontos observados nos dados:**

- O melhor resultado ocorre na **época 4** e não é superado nas 10 épocas seguintes. A execução de 14 épocas (4 + 10) é compatível com um *early stopping* com paciência de 10; confirme na sua configuração.
- A **acurácia de treino estagna em ≈ 60%** a partir da época 3, e a loss de treino fica em torno de 1,10. O treino praticamente não evolui ao longo das épocas.
- A **acurácia de validação oscila muito** (65,9% a 72,0%, amplitude de ≈ 6,1 p.p.), mais do que no Teste 1.
- A validação supera o treino em todas as épocas. Isso é compatível com o augmentation ser aplicado apenas ao treino (confirmado na configuração), o que o torna mais difícil que a validação. Como o mesmo ocorre no Teste 1, sem augmentation, essa não deve ser a única explicação.
- O tempo por época (≈ 4,5 min) é maior que no Teste 1 (≈ 3,5 min), compatível com o custo de aplicar transformações a cada lote.

**Hipóteses a verificar (não confirmadas pelos dados):**

- O `D_pesado` combina várias perturbações (escala de ±10%, translação de até 6%, rotação de ±3°, *blur* e apagamento de pequenos trechos). A escala e o apagamento podem alterar ou ocultar morfologia relevante do ECG (formato das ondas, intervalos RR), dificultando a aprendizagem das classes. Testar cada transformação isoladamente (ablação) mostraria qual delas pesa mais.
- Como o treino já estagnava no Teste 1, o problema pode não ser a falta de variedade de dados, e sim o *setup* de treinamento (learning rate, camadas congeladas, scheduler).

## Comparação com o Teste 1

| Métrica | Teste 1 (`A_sem_aug`) | Teste 4 (`D_pesado`) | Diferença (T4 − T1) |
|---|---:|---:|---:|
| Acurácia de validação (melhor) | 73,70% | 71,99% | −1,71 p.p. |
| Loss de validação (melhor) | 0,7592 | 0,8382 | +0,0790 |
| Acurácia de treino (melhor época) | 67,24% | 60,40% | −6,84 p.p. |
| Loss de treino (melhor época) | 0,9214 | 1,0927 | +0,1713 |
| Tempo de execução | 49,58 min | 63,58 min | +14,00 min (+28%) |

Nestes dois experimentos, o augmentation pesado **não trouxe ganho** nas métricas de validação e aumentou o tempo de treinamento. A diferença de 1,71 p.p. é **menor que a oscilação entre épocas** e cada teste foi rodado uma única vez, portanto **não deve ser tratada como diferença estatisticamente comprovada**. Também não é possível atribuir o resultado exclusivamente ao augmentation sem controlar as demais condições (seed, divisão dos dados, hiperparâmetros).

## Limitações

- Apenas **uma execução** por configuração; sem repetições não há estimativa de variância.
- As métricas são de **treino e validação**. Não há resultado em conjunto de teste independente.
- Só há acurácia global; falta análise **por classe**.
- A "melhor época" é escolhida com a própria validação, o que pode superestimar o desempenho real.

## Próximos passos

- [ ] Avaliar o melhor modelo (época 4) em um conjunto de **teste independente**.
- [ ] Gerar **matriz de confusão** e **precisão, revocação e F1 por classe**.
- [ ] Repetir com 3 a 5 seeds e reportar média ± desvio-padrão.
- [ ] Testar níveis intermediários de augmentation (leve e moderado) para ver se há ponto ótimo.
- [ ] Visualizar amostras aumentadas para verificar se o ECG continua legível e fisiologicamente plausível.
- [ ] Preencher a tabela de informações de reprodutibilidade acima.

## Conclusão

A VGG16 com **augmentation pesado (`D_pesado`)** atingiu **71,99% de acurácia de validação** e **loss de validação de 0,8382** na época 4, em 63,58 minutos. Em relação ao baseline sem augmentation (73,70%), o resultado foi ligeiramente inferior e mais lento, sem evidência de que a estratégia tenha melhorado a generalização neste setup. A conclusão definitiva depende de avaliação em teste independente, métricas por classe e repetições do experimento.