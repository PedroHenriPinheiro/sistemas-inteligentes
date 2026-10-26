# ResNet50 com Data Augmentation Leve

> Experimento da série de testes com ResNet50 para classificação de ritmos cardíacos a partir de sinais de ECG representados como imagens, usando a estratégia de augmentation `B_leve`.

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

Avaliar a **ResNet50** na classificação de sinais de ECG aplicando **data augmentation leve** (`B_leve`), o nível mais fraco de augmentation da série. O resultado é comparado ao baseline sem augmentation da mesma arquitetura (`A_sem_aug`) e ao Teste 2 da VGG16, que usou a mesma estratégia.

## Configuração do experimento

| Parâmetro | Valor |
|---|---|
| Arquitetura | ResNet50 |
| Estratégia de augmentation | `B_leve`: augmentation leve |
| Número de classes | 6 |
| Épocas executadas | 36 |
| Melhor época | 26 |
| Critério de seleção | Menor loss de validação |
| Early stopping | Compatível com paciência de 10 épocas (melhor época 26 + 10 = 36 épocas executadas) |
| Tempo total de execução | 123,15 min (≈ 3,4 min/época) |

**Classes:** `AF`, `AFIB`, `SA`, `SB`, `SR`, `ST`

### Transformações do augmentation `B_leve`

| Transformação | Parâmetros |
|---|---|
| `Resize` | 224 × 224 |
| `RandomAffine` | `degrees=0`, `translate=(0.02, 0.02)`, `fill=(255, 255, 255)` (translação de até 2% em cada eixo; áreas novas em branco) |
| `ColorJitter` | `brightness=0.1` |
| `ToTensor` + `Normalize` | média `[0.485, 0.456, 0.406]`, desvio `[0.229, 0.224, 0.225]` |

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
| Acurácia | 76,16% | **78,07%** |
| Loss | 0,6760 | **0,6229** |

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
| 1 | 0,8939 | 0,7682 | 69,45% | 72,84% |
| 2 | 0,7595 | 0,6989 | 73,67% | 75,47% |
| 3 | 0,7270 | 0,6803 | 74,45% | 76,76% |
| 4 | 0,7103 | 0,6679 | 75,18% | 76,67% |
| 5 | 0,6998 | 0,6542 | 75,36% | 77,63% |
| 6 | 0,6990 | 0,6526 | 75,62% | 77,37% |
| 7 | 0,6889 | 0,6561 | 76,04% | 77,31% |
| 8 | 0,6830 | 0,6395 | 75,97% | 77,56% |
| 9 | 0,6865 | 0,6502 | 76,08% | 76,79% |
| 10 | 0,6816 | 0,6550 | 76,12% | 77,22% |
| 11 | 0,6713 | 0,6334 | 76,22% | 77,83% |
| 12 | 0,6804 | 0,6479 | 76,16% | 77,61% |
| 13 | 0,6769 | 0,6343 | 76,11% | 77,75% |
| 14 | 0,6694 | 0,6400 | 76,24% | 77,77% |
| 15 | 0,6675 | 0,6450 | 76,50% | 77,50% |
| 16 | 0,6738 | 0,6424 | 76,10% | 77,47% |
| 17 | 0,6725 | 0,6343 | 76,31% | 78,13% |
| 18 | 0,6620 | 0,6412 | 76,55% | 77,58% |
| 19 | 0,6690 | 0,6338 | 76,32% | 77,53% |
| 20 | 0,6704 | 0,6397 | 76,49% | 77,48% |
| 21 | 0,6661 | 0,6319 | 76,44% | 78,37% |
| 22 | 0,6595 | 0,6422 | 77,03% | 77,41% |
| 23 | 0,6666 | 0,6499 | 76,62% | 77,45% |
| 24 | 0,6692 | 0,6267 | 76,32% | 78,05% |
| 25 | 0,6696 | 0,6354 | 76,46% | 78,22% |
| **26** | 0,6760 | **0,6229** | 76,16% | **78,07%** |
| 27 | 0,6704 | 0,6559 | 76,20% | 77,01% |
| 28 | 0,6681 | 0,6323 | 76,39% | 77,81% |
| 29 | 0,6684 | 0,6243 | 76,41% | 78,11% |
| 30 | 0,6659 | 0,6412 | 76,47% | 77,92% |
| 31 | 0,6673 | 0,6318 | 76,50% | 78,37% |
| 32 | 0,6651 | 0,6455 | 76,58% | 77,41% |
| 33 | 0,6664 | 0,6500 | 76,49% | 77,01% |
| 34 | 0,6693 | 0,6299 | 76,29% | 77,64% |
| 35 | 0,6628 | 0,6333 | 76,83% | 77,45% |
| 36 | 0,6678 | 0,6485 | 76,42% | 77,80% |

### Resumo estatístico (36 épocas)

| Métrica | Mínimo | Máximo | Média |
|---|---:|---:|---:|
| Acurácia de validação | 72,84% (ép. 1) | 78,37% (ép. 21 e 31) | 77,40% |
| Acurácia de treino | 69,45% (ép. 1) | 77,03% (ép. 22) | 75,96% |
| Loss de validação | 0,6229 (ép. 26) | 0,7682 (ép. 1) | 0,6477 |
| Loss de treino | 0,6595 (ép. 22) | 0,8939 (ép. 1) | 0,6839 |

## Análise

**Pontos observados nos dados:**

- **O treino aprende rápido e depois estagna.** A acurácia de treino sobe de 69,45% (época 1) para 75,36% (época 5) e, da época 7 em diante, fica entre 75,97% e 77,03%. A loss de treino cai de 0,89 para ≈ 0,67.
- **A acurácia de treino estabiliza num patamar que cai com a intensidade do augmentation:** no máximo 77,03% aqui, contra 81,59% no baseline sem augmentation.
- **A validação oscila pouco, mas mais que no baseline.** Da época 4 em diante, a acurácia de validação fica entre 76,67% e 78,37% (amplitude de 1,7 p.p.), contra 1,3 p.p. no baseline.
- **A loss de validação chega a um platô.** Da época 8 em diante ela oscila entre 0,6229 e 0,6559. A melhor época (26) é um ponto desse platô: 7 épocas ficam a menos de 0,01 da melhor loss (épocas 21, 24, 26, 28, 29, 31, 34), e a segunda melhor é a época 29 (0,6243).
- **A melhor época pela loss não é a de maior acurácia.** A maior acurácia de validação (78,37%) ocorreu nas épocas 21 e 31; a época escolhida (26) tem 78,07%, 0,30 p.p. a menos.
- **A validação fica acima do treino em todas as 36 épocas.** Na melhor época, a diferença é de +1,91 p.p. e, na última, +1,38 p.p. No baseline sem augmentation, o treino ultrapassa a validação a partir da época 10, e aqui isso não acontece. É compatível com o augmentation, aplicado só no treino (confirmado na configuração), tornar o treino mais difícil que a validação.
- **O early stopping encerrou o treino na época 36, 10 épocas depois do melhor resultado (época 26).** É compatível com paciência de 10; confirme na sua configuração.

**Hipóteses a verificar (não confirmadas pelos dados):**

- A queda frente ao baseline pode vir de o augmentation leve ainda perturbar a posição dos traçados (translação) e o brilho, o que dificultaria o ajuste fino. Só um experimento controlado, com seeds repetidas, mostra se o efeito é real.
- Sem o relatório por classe, não dá para saber se a perda de acurácia global vem das classes frequentes ou das raras (`AFIB`, `SA`), que na VGG16 já eram as mais problemáticas.

## Comparação com os demais testes

### Entre os níveis de augmentation da ResNet50

Métricas na melhor época (selecionada pela menor loss de validação):

| Métrica | `A_sem_aug` | **`B_leve`** | `C_moderado` | `D_pesado` |
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
- **Frente ao baseline `A_sem_aug`**, o `B_leve` teve −2,25 p.p. de acurácia de validação, +0,0758 de loss de validação e −4,53 p.p. de acurácia de treino.
- **O `B_leve` ficou à frente do `C_moderado`** por 0,80 p.p. na melhor época e 1,17 p.p. na média das épocas. Essa diferença é menor que a oscilação da validação (≈ 2,0 p.p.), então não permite separar os dois níveis com segurança.
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
- **No `B_leve`**, a ResNet50 teve +6,28 p.p. de acurácia de validação e −0,1611 de loss de validação em relação ao Teste 2 da VGG16.
- **Por época, a ResNet50 foi mais rápida que a VGG16 em todos os níveis** (3,42 contra 3,93 min no `B_leve`). O tempo total da ResNet50 foi maior porque ela rodou mais épocas (36 contra 23).

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
- [ ] Visualizar amostras aumentadas pelo `B_leve` para verificar se o ECG continua legível e fisiologicamente plausível.
- [ ] Considerar a **média das últimas épocas** ou o **F1 macro** como critério de seleção do modelo, já que a loss de validação oscila.
- [ ] Testar **pesos de classe**, `WeightedRandomSampler` ou *focal loss* para o desbalanceamento.
- [ ] Repetir com 3 a 5 seeds e reportar média ± desvio-padrão.
- [ ] Preencher a tabela de reprodutibilidade.

## Conclusão

A ResNet50 com **augmentation leve (`B_leve`)** atingiu **78,07% de acurácia de validação** e **loss de validação de 0,6229** na época 26, em 123,15 minutos (36 épocas). Em relação ao baseline sem augmentation (80,32%), o resultado foi 2,25 p.p. inferior, e o treino precisou de mais épocas e de mais tempo. Mesmo assim, ficou 6,28 p.p. acima do mesmo nível da VGG16. A conclusão definitiva depende de relatório por classe, teste independente e repetições do experimento.