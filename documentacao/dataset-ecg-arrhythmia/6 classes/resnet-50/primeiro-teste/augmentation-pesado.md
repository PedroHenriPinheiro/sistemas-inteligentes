# ResNet50 com Data Augmentation Pesado

> Experimento da série de testes com ResNet50 para classificação de ritmos cardíacos a partir de sinais de ECG representados como imagens, usando a estratégia de augmentation `D_pesado`.

## Sumário

1. [Objetivo](#objetivo)
2. [Configuração do experimento](#configuração-do-experimento)
3. [Resultados da melhor época](#resultados-da-melhor-época)
4. [Desempenho por classe](#desempenho-por-classe)
5. [Histórico de treinamento](#histórico-de-treinamento)
6. [Análise](#análise)
7. [Comparação com os demais testes](#comparação-com-os-demais-testes)
8. [Limitações](#limitações)
9. [Próximos passos](#próximos-passos)
10. [Conclusão](#conclusão)

## Objetivo

Avaliar a **ResNet50** na classificação de sinais de ECG aplicando **data augmentation pesado** (`D_pesado`), o nível mais forte da série. Com este teste, a ResNet50 cobre os quatro níveis de intensidade (nenhum, leve, moderado e pesado), como a VGG16. O resultado é comparado aos demais níveis da ResNet50 e ao Teste 4 da VGG16.

## Configuração do experimento

| Parâmetro | Valor |
|---|---|
| Arquitetura | ResNet50 |
| Estratégia de augmentation | `D_pesado`: augmentation pesado |
| Número de classes | 6 |
| Épocas executadas | 36 |
| Melhor época | 26 |
| Critério de seleção | Menor loss de validação |
| Early stopping | Compatível com paciência de 10 épocas (melhor época 26 + 10 = 36 épocas executadas) |
| Tempo total de execução | 156,37 min (≈ 4,3 min/época) |

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
| Camadas congeladas / *fine-tuning* | _a preencher_ |
| Otimizador e *learning rate* | _a preencher_ |
| Batch size | _a preencher_ |
| Tratamento de desbalanceamento (pesos de classe, amostragem) | _a preencher_ |
| Seed | _a preencher_ |
| GPU / ambiente | _a preencher_ |

> Os hiperparâmetros precisam ser iguais aos do baseline `A_sem_aug` da ResNet50 para a comparação entre níveis de augmentation ser justa. Se forem diferentes, registre as diferenças aqui.

## Resultados da melhor época

Melhor época: **26**, com a menor loss de validação do experimento.

| Métrica | Treino | Validação |
|---|---:|---:|
| Acurácia | 71,93% | **73,89%** |
| Loss | 0,7885 | **0,7208** |

## Desempenho por classe

> **Pendente.** O JSON deste teste não inclui o relatório de classificação. Para completar, envie a saída de `classification_report` (validação e teste) do melhor modelo, de preferência com `zero_division=0`.

| Classe | Precisão | Revocação | F1 | Suporte |
|---|---:|---:|---:|---:|
| AF | _a preencher_ | _a preencher_ | _a preencher_ | — |
| AFIB | _a preencher_ | _a preencher_ | _a preencher_ | — |
| SA | _a preencher_ | _a preencher_ | _a preencher_ | — |
| SB | _a preencher_ | _a preencher_ | _a preencher_ | — |
| SR | _a preencher_ | _a preencher_ | _a preencher_ | — |
| ST | _a preencher_ | _a preencher_ | _a preencher_ | — |
| **Acurácia (teste)** | | | _a preencher_ | |
| **F1 macro (teste)** | | | _a preencher_ | |

## Histórico de treinamento

| Época | Loss treino | Loss val. | Acc. treino | Acc. val. |
|---:|---:|---:|---:|---:|
| 1 | 1,0115 | 0,9076 | 64,56% | 68,16% |
| 2 | 0,8782 | 0,8609 | 69,07% | 69,66% |
| 3 | 0,8408 | 0,7929 | 70,53% | 72,57% |
| 4 | 0,8328 | 0,7797 | 70,45% | 72,63% |
| 5 | 0,8220 | 0,7899 | 71,01% | 72,48% |
| 6 | 0,8198 | 0,7600 | 71,13% | 73,80% |
| 7 | 0,8080 | 0,7593 | 71,24% | 73,37% |
| 8 | 0,8082 | 0,7465 | 71,46% | 73,80% |
| 9 | 0,8045 | 0,7696 | 71,24% | 72,73% |
| 10 | 0,8031 | 0,7701 | 71,45% | 72,82% |
| 11 | 0,7940 | 0,7516 | 71,63% | 73,42% |
| 12 | 0,8028 | 0,7438 | 71,48% | 73,85% |
| 13 | 0,8039 | 0,7321 | 71,54% | 74,60% |
| 14 | 0,7989 | 0,7348 | 71,81% | 74,51% |
| 15 | 0,8015 | 0,7622 | 71,68% | 74,26% |
| 16 | 0,7992 | 0,7308 | 71,96% | 74,10% |
| 17 | 0,7924 | 0,7338 | 71,90% | 74,26% |
| 18 | 0,7906 | 0,7467 | 71,93% | 73,86% |
| 19 | 0,7926 | 0,7516 | 71,74% | 73,99% |
| 20 | 0,7926 | 0,7343 | 72,04% | 74,13% |
| 21 | 0,7953 | 0,7395 | 71,99% | 74,29% |
| 22 | 0,7862 | 0,7602 | 72,06% | 73,59% |
| 23 | 0,7949 | 0,7440 | 71,93% | 74,35% |
| 24 | 0,7927 | 0,7832 | 71,87% | 74,18% |
| 25 | 0,7971 | 0,7581 | 71,97% | 74,52% |
| **26** | 0,7885 | **0,7208** | 71,93% | **73,89%** |
| 27 | 0,7970 | 0,7402 | 71,41% | 75,12% |
| 28 | 0,7950 | 0,7371 | 71,78% | 74,59% |
| 29 | 0,7932 | 0,7385 | 71,88% | 74,82% |
| 30 | 0,7900 | 0,7462 | 71,60% | 74,24% |
| 31 | 0,7905 | 0,7471 | 71,93% | 73,86% |
| 32 | 0,7930 | 0,7696 | 71,70% | 74,74% |
| 33 | 0,7978 | 0,7579 | 71,70% | 74,05% |
| 34 | 0,7917 | 0,7217 | 71,86% | 75,03% |
| 35 | 0,7934 | 0,7434 | 72,04% | 74,71% |
| 36 | 0,7907 | 0,7649 | 71,93% | 73,52% |

### Resumo estatístico (36 épocas)

| Métrica | Mínimo | Máximo | Média |
|---|---:|---:|---:|
| Acurácia de validação | 68,16% (ép. 1) | 75,12% (ép. 27) | 73,68% |
| Acurácia de treino | 64,56% (ép. 1) | 72,06% (ép. 22) | 71,37% |
| Loss de validação | 0,7208 (ép. 26) | 0,9076 (ép. 1) | 0,7592 |
| Loss de treino | 0,7862 (ép. 22) | 1,0115 (ép. 1) | 0,8079 |

## Análise

**Pontos observados nos dados:**

- **O treino aprende rápido e depois estagna.** A acurácia de treino sobe de 64,56% (época 1) para 71,01% (época 5) e, da época 5 em diante, fica entre 71,01% e 72,06%. A loss de treino cai de 1,01 para ≈ 0,79.
- **A acurácia de treino estabiliza num patamar que cai com a intensidade do augmentation:** no máximo 72,06% aqui, contra 81,59% no baseline sem augmentation.
- **A validação oscila pouco, mas mais que no baseline.** Da época 4 em diante, a acurácia de validação fica entre 72,48% e 75,12% (amplitude de 2,6 p.p.), contra 1,3 p.p. no baseline.
- **A loss de validação é ruidosa.** Da época 13 em diante ela varia entre 0,7208 e 0,7832. A melhor época (26) é quase um ponto isolado: só 2 épocas estão a menos de 0,01 da melhor loss (épocas 26, 34).
- **A melhor época pela loss não é a de maior acurácia.** A maior acurácia de validação (75,12%) ocorreu na época 27, com loss de 0,7402; a época escolhida (26) tem 73,89%, 1,23 p.p. a menos. Esta é a maior divergência entre os dois critérios nos testes da ResNet50.
- **A validação fica acima do treino em todas as 36 épocas.** Na melhor época, a diferença é de +1,96 p.p. e, na última, +1,59 p.p. No baseline sem augmentation, o treino ultrapassa a validação a partir da época 10, e aqui isso não acontece. É compatível com o augmentation, aplicado só no treino (confirmado na configuração), tornar o treino mais difícil que a validação.
- **O early stopping encerrou o treino na época 36, 10 épocas depois do melhor resultado (época 26).** É compatível com paciência de 10; confirme na sua configuração.

**Hipóteses a verificar (não confirmadas pelos dados):**

- O `D_pesado` combina várias perturbações (escala de ±10%, translação de até 6%, rotação de ±3°, *blur* e apagamento de pequenos trechos). A escala e o apagamento podem alterar ou ocultar morfologia relevante do ECG (formato das ondas, intervalos RR). Testar cada transformação isoladamente (ablação) mostraria qual delas pesa mais.
- Sem o relatório por classe, não dá para saber se a perda de acurácia global vem das classes frequentes ou das raras (`AFIB`, `SA`), que na VGG16 já eram as mais problemáticas.

## Comparação com os demais testes

### Entre os níveis de augmentation da ResNet50

Métricas na melhor época (selecionada pela menor loss de validação):

| Métrica | `A_sem_aug` | `B_leve` | `C_moderado` | **`D_pesado`** |
|---|---:|---:|---:|---:|
| Acurácia de validação (melhor época) | 80,32% | 78,07% | 77,26% | 73,89% |
| Acurácia de validação (média das épocas) | 80,32% | 77,40% | 76,23% | 73,68% |
| Acurácia de validação (média das últimas 10 épocas) | 80,54% | 77,65% | 76,56% | 74,47% |
| Loss de validação | 0,5471 | 0,6229 | 0,6550 | 0,7208 |
| Acurácia de treino | 80,69% | 76,16% | 74,42% | 71,93% |
| Loss de treino | 0,5391 | 0,6760 | 0,7238 | 0,7885 |
| Validação − treino (acurácia, melhor época) | −0,38 p.p. | +1,91 p.p. | +2,85 p.p. | +1,96 p.p. |
| Faixa da acurácia de validação (épocas 4 em diante) | 80,14% a 81,42% (1,3 p.p.) | 76,67% a 78,37% (1,7 p.p.) | 75,26% a 77,26% (2,0 p.p.) | 72,48% a 75,12% (2,6 p.p.) |
| Melhor época / épocas executadas | 10 / 20 | 26 / 36 | 27 / 37 | 26 / 36 |
| Tempo total | 63,97 min | 123,15 min | 136,39 min | 156,37 min |
| Tempo médio por época | 3,20 min | 3,42 min | 3,69 min | 4,34 min |

**O que a tabela mostra:**

- **A ordem é a mesma em todas as métricas de validação e treino:** quanto mais augmentation, menor a acurácia de validação na melhor época (80,32% → 78,07% → 77,26% → 73,89%), maior a loss de validação (0,547 → 0,623 → 0,655 → 0,721) e menor a acurácia de treino (80,7% → 76,2% → 74,4% → 71,9%). Isso difere da VGG16, em que a acurácia de validação na melhor época não seguia a ordem dos níveis.
- **A oscilação da validação cresce com o augmentation:** a amplitude da acurácia de validação, da época 4 em diante, vai de 1,3 p.p. (`A_sem_aug`) a 2,6 p.p. (`D_pesado`).
- **Frente ao baseline `A_sem_aug`**, o `D_pesado` teve −6,42 p.p. de acurácia de validação, +0,1737 de loss de validação e −8,76 p.p. de acurácia de treino.
- **Frente ao `C_moderado`**, o `D_pesado` teve −3,37 p.p. na melhor época, diferença maior que a oscilação da validação (≈ 2,6 p.p.), e −2,55 p.p. na média das épocas, da mesma ordem da oscilação. É a maior queda entre dois níveis consecutivos na melhor época.
- **O custo de tempo cresce com o augmentation**, tanto por época (3,20 → 3,42 → 3,69 → 4,34 min) quanto no total (64,0 → 123,2 → 136,4 → 156,4 min). Os níveis com augmentation precisaram de 36, 37 e 36 épocas, contra 20 do baseline, o que explica a maior parte do tempo total. O ambiente de execução pode ter diferido entre os testes.

### ResNet50 × VGG16 no mesmo nível de augmentation

| Estratégia | Acc. val. ResNet50 | Acc. val. VGG16 | Diferença | Loss val. ResNet50 | Loss val. VGG16 | Diferença | Min/época ResNet50 | Min/época VGG16 |
|---|---:|---:|---:|---:|---:|---:|---:|---:|
| `A_sem_aug` (VGG16 Teste 1) | 80,32% | 73,70% | +6,61 p.p. | 0,5471 | 0,7592 | −0,2121 | 3,20 | 3,54 |
| `B_leve` (VGG16 Teste 2) | 78,07% | 71,78% | +6,28 p.p. | 0,6229 | 0,7840 | −0,1611 | 3,42 | 3,93 |
| `C_moderado` (VGG16 Teste 3) | 77,26% | 73,22% | +4,05 p.p. | 0,6550 | 0,8054 | −0,1504 | 3,69 | 3,95 |
| `D_pesado` (VGG16 Teste 4) | 73,89% | 71,99% | +1,91 p.p. | 0,7208 | 0,8382 | −0,1174 | 4,34 | 4,54 |

**O que a tabela mostra:**

- **A ResNet50 ficou à frente da VGG16 em todos os níveis**, tanto em acurácia quanto em loss de validação. A vantagem em acurácia diminui com o augmentation: +6,61 → +6,28 → +4,05 → +1,91 p.p. (`A_sem_aug` → `D_pesado`).
- **No `D_pesado`**, a ResNet50 teve +1,91 p.p. de acurácia de validação e −0,1174 de loss de validação em relação ao Teste 4 da VGG16.
- **Mesmo com o augmentation mais forte, a ResNet50 (73,89%) igualou o melhor resultado de acurácia de validação da VGG16 (73,70%, sem augmentation)**, com diferença de +0,19 p.p., dentro da oscilação. A loss de validação da ResNet50 (0,7208) é menor que a de qualquer teste da VGG16 (menor valor da VGG16: 0,7592).
- **Por época, a ResNet50 foi mais rápida que a VGG16 em todos os níveis** (4,34 contra 4,54 min no `D_pesado`). O tempo total da ResNet50 foi maior porque ela rodou mais épocas (36 contra 14).

**Ressalva:** cada configuração foi executada uma única vez, e a comparação entre arquiteturas só é rigorosa se dataset, divisão dos dados, hiperparâmetros, pesos iniciais e seed forem os mesmos. Só há métricas de treino e validação; a comparação por classe e em conjunto de teste independente ainda não pode ser feita. As diferenças pequenas (menores que a oscilação entre épocas) não devem ser tratadas como estatisticamente comprovadas.

## Limitações

- Apenas **uma execução** por configuração; não há estimativa de variância (seeds diferentes).
- As métricas são de **treino e validação**. Não há resultado em conjunto de teste independente.
- Só há acurácia global. Faltam **relatório por classe** e **matriz de confusão**. O Teste 2 da VGG16 mostrou que a acurácia global pode esconder classes não reconhecidas (`AFIB`, `SA`).
- A "melhor época" é selecionada com a própria validação, e a loss de validação oscila entre épocas, o que torna a escolha sensível a ruído.
- Hiperparâmetros, pesos iniciais, dataset e divisão dos dados não estão documentados, o que limita a comparação entre arquiteturas.

## Próximos passos

- [ ] Gerar o **relatório de classificação** (validação e teste) e a **matriz de confusão** do melhor modelo (época 26), com `zero_division=0`.
- [ ] Avaliar em um conjunto de **teste independente** e comparar os quatro níveis da ResNet50 por **F1 macro**.
- [ ] Confirmar que hiperparâmetros, divisão dos dados e pesos iniciais são iguais aos do baseline `A_sem_aug`.
- [ ] Fazer **ablação** das transformações do `D_pesado` (escala, rotação, *blur*, apagamento) para ver qual mais prejudica o desempenho.
- [ ] Considerar a **média das últimas épocas** ou o **F1 macro** como critério de seleção do modelo, já que a loss de validação oscila.
- [ ] Testar **pesos de classe**, `WeightedRandomSampler` ou *focal loss* para o desbalanceamento.
- [ ] Repetir com 3 a 5 seeds e reportar média ± desvio-padrão.
- [ ] Preencher a tabela de reprodutibilidade.

## Conclusão

A ResNet50 com **augmentation pesado (`D_pesado`)** atingiu **73,89% de acurácia de validação** e **loss de validação de 0,7208** na época 26, em 156,37 minutos (36 épocas), o pior resultado entre os quatro níveis da ResNet50 e o mais lento. Comparando os quatro níveis, quanto maior o augmentation, pior o desempenho de validação e de treino, sem evidência de ganho de generalização neste setup. Ainda assim, o resultado equivale ao melhor da VGG16 sem augmentation. A conclusão definitiva depende de relatório por classe, teste independente e repetições do experimento.