# ResNet50 com Data Augmentation Moderado

> Experimento da série de testes com ResNet50 para classificação de ritmos cardíacos a partir de sinais de ECG representados como imagens, usando a estratégia de augmentation `C_moderado`.

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

Avaliar a **ResNet50** na classificação de sinais de ECG aplicando **data augmentation moderado** (`C_moderado`), nível intermediário entre o leve (`B_leve`) e o pesado (`D_pesado`). O resultado é comparado aos demais níveis da ResNet50 e ao Teste 3 da VGG16, que usou a mesma estratégia.

## Configuração do experimento

| Parâmetro | Valor |
|---|---|
| Arquitetura | ResNet50 |
| Estratégia de augmentation | `C_moderado`: augmentation moderado |
| Número de classes | 6 |
| Épocas executadas | 37 |
| Melhor época | 27 |
| Critério de seleção | Menor loss de validação |
| Early stopping | Compatível com paciência de 10 épocas (melhor época 27 + 10 = 37 épocas executadas) |
| Tempo total de execução | 136,39 min (≈ 3,7 min/época) |

**Classes:** `AF`, `AFIB`, `SA`, `SB`, `SR`, `ST`

### Transformações do augmentation `C_moderado`

| Transformação | Parâmetros |
|---|---|
| `Resize` | 224 × 224 |
| `RandomAffine` | `degrees=0`, `translate=(0.04, 0.04)`, `scale=(0.95, 1.05)`, `fill=(255, 255, 255)` (translação de até 4% e escala de ±5%; áreas novas em branco) |
| `ColorJitter` | `brightness=0.2`, `contrast=0.2` |
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

Melhor época: **27**, com a menor loss de validação do experimento. Neste teste, a época 27 também é a de **maior acurácia de validação** (77,26%), ou seja, os dois critérios apontam para o mesmo modelo.

| Métrica | Treino | Validação |
|---|---:|---:|
| Acurácia | 74,42% | **77,26%** |
| Loss | 0,7238 | **0,6550** |

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
| 1 | 0,9331 | 0,8153 | 67,61% | 71,64% |
| 2 | 0,8000 | 0,7632 | 72,12% | 74,02% |
| 3 | 0,7683 | 0,7305 | 73,03% | 75,44% |
| 4 | 0,7595 | 0,7155 | 73,43% | 75,88% |
| 5 | 0,7492 | 0,7129 | 73,44% | 76,10% |
| 6 | 0,7437 | 0,6900 | 73,99% | 76,65% |
| 7 | 0,7404 | 0,6969 | 74,16% | 76,11% |
| 8 | 0,7383 | 0,6904 | 73,83% | 76,35% |
| 9 | 0,7410 | 0,7040 | 73,91% | 75,74% |
| 10 | 0,7289 | 0,7044 | 74,48% | 75,26% |
| 11 | 0,7255 | 0,6812 | 74,44% | 75,70% |
| 12 | 0,7302 | 0,6834 | 74,40% | 76,78% |
| 13 | 0,7340 | 0,6734 | 74,11% | 77,22% |
| 14 | 0,7256 | 0,6767 | 74,42% | 77,14% |
| 15 | 0,7215 | 0,6871 | 74,62% | 76,67% |
| 16 | 0,7242 | 0,6618 | 74,51% | 76,98% |
| 17 | 0,7262 | 0,6760 | 74,38% | 77,23% |
| 18 | 0,7226 | 0,6955 | 74,40% | 76,30% |
| 19 | 0,7244 | 0,6914 | 74,45% | 76,15% |
| 20 | 0,7197 | 0,6801 | 74,55% | 76,78% |
| 21 | 0,7241 | 0,6728 | 74,10% | 76,29% |
| 22 | 0,7190 | 0,6917 | 74,63% | 75,93% |
| 23 | 0,7254 | 0,7050 | 74,37% | 76,46% |
| 24 | 0,7218 | 0,6843 | 74,38% | 76,79% |
| 25 | 0,7282 | 0,6928 | 73,94% | 76,00% |
| 26 | 0,7208 | 0,6572 | 74,42% | 76,02% |
| **27** | 0,7238 | **0,6550** | 74,42% | **77,26%** |
| 28 | 0,7246 | 0,6692 | 74,53% | 76,49% |
| 29 | 0,7129 | 0,6801 | 74,83% | 76,37% |
| 30 | 0,7180 | 0,6770 | 74,75% | 76,27% |
| 31 | 0,7235 | 0,6662 | 74,56% | 76,73% |
| 32 | 0,7213 | 0,6731 | 74,61% | 76,87% |
| 33 | 0,7332 | 0,6756 | 74,08% | 76,89% |
| 34 | 0,7204 | 0,6705 | 74,58% | 77,00% |
| 35 | 0,7260 | 0,6820 | 74,18% | 76,87% |
| 36 | 0,7235 | 0,6915 | 74,36% | 76,18% |
| 37 | 0,7221 | 0,6993 | 74,39% | 75,97% |

### Resumo estatístico (37 épocas)

| Métrica | Mínimo | Máximo | Média |
|---|---:|---:|---:|
| Acurácia de validação | 71,64% (ép. 1) | 77,26% (ép. 27) | 76,23% |
| Acurácia de treino | 67,61% (ép. 1) | 74,83% (ép. 29) | 74,04% |
| Loss de validação | 0,6550 (ép. 27) | 0,8153 (ép. 1) | 0,6912 |
| Loss de treino | 0,7129 (ép. 29) | 0,9331 (ép. 1) | 0,7363 |

## Análise

**Pontos observados nos dados:**

- **O treino aprende rápido e depois estagna.** A acurácia de treino sobe de 67,61% (época 1) para 73,44% (época 5) e, da época 6 em diante, fica entre 73,83% e 74,83%. A loss de treino cai de 0,93 para ≈ 0,72.
- **A acurácia de treino estabiliza num patamar que cai com a intensidade do augmentation:** no máximo 74,83% aqui, contra 81,59% no baseline sem augmentation.
- **A validação oscila pouco, mas mais que no baseline.** Da época 4 em diante, a acurácia de validação fica entre 75,26% e 77,26% (amplitude de 2,0 p.p.), contra 1,3 p.p. no baseline.
- **A loss de validação oscila muito antes de melhorar.** Da época 11 em diante ela varia entre 0,6550 e 0,7050. A melhor época (27) fica isolada: só 3 épocas estão a menos de 0,01 da melhor loss (épocas 16, 26, 27).
- **Aqui os dois critérios coincidem.** A época 27 tem ao mesmo tempo a menor loss e a maior acurácia de validação (77,26%). Ela está 0,70 p.p. acima da média das últimas 10 épocas (76,56%), ou seja, é um pico moderado.
- **A validação fica acima do treino em todas as 37 épocas.** Na melhor época, a diferença é de +2,85 p.p. e, na última, +1,58 p.p. No baseline sem augmentation, o treino ultrapassa a validação a partir da época 10, e aqui isso não acontece. É compatível com o augmentation, aplicado só no treino (confirmado na configuração), tornar o treino mais difícil que a validação.
- **O early stopping encerrou o treino na época 37, 10 épocas depois do melhor resultado (época 27).** É compatível com paciência de 10; confirme na sua configuração.

**Hipóteses a verificar (não confirmadas pelos dados):**

- A escala de ±5% pode deformar levemente os intervalos entre ondas (por exemplo, o intervalo RR), que são informação central em ritmos como `AF` e `AFIB`. Testar o `C_moderado` sem escala ajudaria a isolar esse efeito.
- Sem o relatório por classe, não dá para saber se a perda de acurácia global vem das classes frequentes ou das raras (`AFIB`, `SA`), que na VGG16 já eram as mais problemáticas.

## Comparação com os demais testes

### Entre os níveis de augmentation da ResNet50

Métricas na melhor época (selecionada pela menor loss de validação):

| Métrica | `A_sem_aug` | `B_leve` | **`C_moderado`** | `D_pesado` |
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
- **Frente ao baseline `A_sem_aug`**, o `C_moderado` teve −3,05 p.p. de acurácia de validação, +0,1079 de loss de validação e −6,28 p.p. de acurácia de treino.
- **Frente ao `B_leve`**, o `C_moderado` teve −0,80 p.p. na melhor época e −1,17 p.p. na média das épocas. Essa diferença é da ordem da oscilação da validação (≈ 2,0 p.p.), então não permite separar os dois níveis com segurança. Já o `D_pesado` fica 3,37 p.p. abaixo na melhor época, uma diferença maior que a oscilação.
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
- **No `C_moderado`**, a ResNet50 teve +4,05 p.p. de acurácia de validação e −0,1504 de loss de validação em relação ao Teste 3 da VGG16.
- **Por época, a ResNet50 foi mais rápida que a VGG16 em todos os níveis** (3,69 contra 3,95 min no `C_moderado`). O tempo total da ResNet50 foi maior porque ela rodou mais épocas (37 contra 27).

**Ressalva:** cada configuração foi executada uma única vez, e a comparação entre arquiteturas só é rigorosa se dataset, divisão dos dados, hiperparâmetros, pesos iniciais e seed forem os mesmos. Só há métricas de treino e validação; a comparação por classe e em conjunto de teste independente ainda não pode ser feita. As diferenças pequenas (menores que a oscilação entre épocas) não devem ser tratadas como estatisticamente comprovadas.

## Limitações

- Apenas **uma execução** por configuração; não há estimativa de variância (seeds diferentes).
- As métricas são de **treino e validação**. Não há resultado em conjunto de teste independente.
- Só há acurácia global. Faltam **relatório por classe** e **matriz de confusão**. O Teste 2 da VGG16 mostrou que a acurácia global pode esconder classes não reconhecidas (`AFIB`, `SA`).
- A "melhor época" é selecionada com a própria validação, e a loss de validação oscila entre épocas, o que torna a escolha sensível a ruído.
- Hiperparâmetros, pesos iniciais, dataset e divisão dos dados não estão documentados, o que limita a comparação entre arquiteturas.

## Próximos passos

- [ ] Gerar o **relatório de classificação** (validação e teste) e a **matriz de confusão** do melhor modelo (época 27), com `zero_division=0`.
- [ ] Avaliar em um conjunto de **teste independente** e comparar os quatro níveis da ResNet50 por **F1 macro**.
- [ ] Confirmar que hiperparâmetros, divisão dos dados e pesos iniciais são iguais aos do baseline `A_sem_aug`.
- [ ] Testar o `C_moderado` sem a escala (apenas translação e cor) para isolar o efeito da escala.
- [ ] Considerar a **média das últimas épocas** ou o **F1 macro** como critério de seleção do modelo, já que a loss de validação oscila.
- [ ] Testar **pesos de classe**, `WeightedRandomSampler` ou *focal loss* para o desbalanceamento.
- [ ] Repetir com 3 a 5 seeds e reportar média ± desvio-padrão.
- [ ] Preencher a tabela de reprodutibilidade.

## Conclusão

A ResNet50 com **augmentation moderado (`C_moderado`)** atingiu **77,26% de acurácia de validação** e **loss de validação de 0,6550** na época 27, em 136,39 minutos (37 épocas). O resultado ficou 3,05 p.p. abaixo do baseline sem augmentation e muito próximo do `B_leve` (0,80 p.p. de diferença, dentro da oscilação). Frente à VGG16 no mesmo nível, a ResNet50 teve 4,05 p.p. a mais. A conclusão definitiva depende de relatório por classe, teste independente e repetições do experimento.