# ResNet50 sem Data Augmentation

> Experimento de referência (*baseline*) da ResNet50 para classificação de ritmos cardíacos a partir de sinais de ECG representados como imagens. Segue o mesmo protocolo da série com VGG16 (testes 1 a 4), o que permite comparar as duas arquiteturas.

## Sumário

1. [Objetivo](#objetivo)
2. [Configuração do experimento](#configuração-do-experimento)
3. [Resultados da melhor época](#resultados-da-melhor-época)
4. [Histórico de treinamento](#histórico-de-treinamento)
5. [Análise](#análise)
6. [Comparação com a VGG16](#comparação-com-a-vgg16)
7. [Limitações](#limitações)
8. [Próximos passos](#próximos-passos)
9. [Conclusão](#conclusão)

## Objetivo

Avaliar o desempenho da arquitetura **ResNet50** na classificação de sinais de ECG **sem aplicar data augmentation** (`A_sem_aug`). Este experimento serve de referência para a ResNet50 e permite comparar a arquitetura com a VGG16 sob a mesma estratégia de augmentation (Teste 1 da série VGG16).

## Configuração do experimento

| Parâmetro | Valor |
|---|---|
| Arquitetura | ResNet50 |
| Estratégia de augmentation | `A_sem_aug`: nenhuma |
| Número de classes | 6 |
| Épocas executadas | 20 |
| Melhor época | 10 |
| Critério de seleção | Menor loss de validação |
| Early stopping | Compatível com paciência de 10 épocas (melhor época 10 + 10 = 20 épocas executadas) |
| Tempo total de execução | 63,97 min (≈ 3,2 min/época) |

**Classes:** `AF`, `AFIB`, `SA`, `SB`, `SR`, `ST`

### Pré-processamento

Sem augmentation, o treino usa o mesmo pipeline de validação e teste (`transform_avaliacao`), padronizado na equipe:

| Transformação | Parâmetros |
|---|---|
| `Resize` | 224 × 224 |
| `ToTensor` + `Normalize` | média `[0.485, 0.456, 0.406]`, desvio `[0.229, 0.224, 0.225]` |

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

> Os hiperparâmetros acima precisam ser iguais aos da série VGG16 para a comparação entre arquiteturas ser justa. Se forem diferentes, registre as diferenças aqui.

## Resultados da melhor época

Melhor época: **10**, com a menor loss de validação do experimento (a época 11 ficou praticamente empatada: 0,5472).

| Métrica | Treino | Validação |
|---|---:|---:|
| Acurácia | 80,69% | **80,32%** |
| Loss | 0,5391 | **0,5471** |

A acurácia de validação da melhor época (80,32%) é igual à **média das 20 épocas** (80,32%). Ou seja, ao contrário do que ocorreu na série VGG16, o resultado reportado não é um pico isolado: representa o desempenho típico do treino. A maior acurácia de validação (81,42%) ocorreu na época 5.

## Histórico de treinamento

| Época | Loss treino | Loss val. | Acc. treino | Acc. val. |
|---:|---:|---:|---:|---:|
| 1 | 0,8095 | 0,6675 | 72,37% | 76,84% |
| 2 | 0,6576 | 0,6155 | 77,44% | 79,42% |
| 3 | 0,6116 | 0,6055 | 78,82% | 79,28% |
| 4 | 0,5906 | 0,5806 | 79,67% | 80,35% |
| 5 | 0,5728 | 0,5581 | 79,94% | **81,42%** |
| 6 | 0,5610 | 0,5492 | 80,06% | 81,07% |
| 7 | 0,5525 | 0,5518 | 80,43% | 80,85% |
| 8 | 0,5464 | 0,5595 | 80,55% | 80,54% |
| 9 | 0,5437 | 0,5524 | 80,52% | 80,82% |
| **10** | 0,5391 | **0,5471** | 80,69% | 80,32% |
| 11 | 0,5231 | 0,5472 | 81,34% | 80,96% |
| 12 | 0,5290 | 0,5626 | 81,01% | 80,37% |
| 13 | 0,5268 | 0,5519 | 81,11% | 81,07% |
| 14 | 0,5205 | 0,5526 | 81,33% | 80,84% |
| 15 | 0,5182 | 0,5559 | 81,24% | 80,74% |
| 16 | 0,5183 | 0,5636 | 81,18% | 80,27% |
| 17 | 0,5229 | 0,5666 | 80,91% | 80,19% |
| 18 | 0,5092 | 0,5551 | 81,59% | 80,59% |
| 19 | 0,5139 | 0,5614 | 81,33% | 80,27% |
| 20 | 0,5104 | 0,5531 | 81,51% | 80,14% |

### Resumo estatístico (20 épocas)

| Métrica | Mínimo | Máximo | Média |
|---|---:|---:|---:|
| Acurácia de validação | 76,84% (ép. 1) | 81,42% (ép. 5) | 80,32% |
| Acurácia de treino | 72,37% (ép. 1) | 81,59% (ép. 18) | 80,15% |
| Loss de validação | 0,5471 (ép. 10) | 0,6675 (ép. 1) | 0,5679 |
| Loss de treino | 0,5092 (ép. 18) | 0,8095 (ép. 1) | 0,5588 |

## Análise

**Pontos observados nos dados:**

- **O treino aprende de forma contínua.** A acurácia de treino sobe de 72,4% (época 1) para ≈ 81% (a partir da época 11), e a loss de treino cai de 0,81 para ≈ 0,51. Isso difere da série VGG16, em que o treino estagnava logo nas primeiras épocas.
- **A validação é estável depois das primeiras épocas.** A partir da época 4, a acurácia de validação fica entre 80,14% e 81,42% (amplitude de ≈ 1,3 p.p.). Na VGG16 sem augmentation, a oscilação entre épocas chegou a ≈ 4,8 p.p.
- **A loss de validação deixa de melhorar na época 10.** Ela cai até 0,5471 e depois fica entre 0,547 e 0,567 até a época 20, enquanto a loss de treino continua caindo (0,539 → 0,510).
- **Começo de distanciamento entre treino e validação.** Nas épocas 1 a 7 e 9, a validação superou o treino. A partir da época 10, o treino passa a ficar sempre acima da validação, e a diferença cresce até 1,4 p.p. na época 20 (81,51% contra 80,14%). Esse comportamento é compatível com o início de sobreajuste, mas a diferença ainda é pequena.
- **A melhor época pela loss não é a de maior acurácia.** A maior acurácia de validação (81,42%) foi na época 5; a época escolhida (10) tem 80,32%. A diferença é de 1,1 p.p.
- **O early stopping encerrou o treino 10 épocas depois do melhor resultado.** A execução de 20 épocas (10 + 10) é compatível com paciência de 10; confirme na sua configuração.

**Hipóteses a verificar (não confirmadas pelos dados):**

- A ResNet50 pode se ajustar melhor aos dados por ter conexões residuais e normalização em lote, o que costuma facilitar a otimização. Porém, a diferença também pode vir de pesos pré-treinados, taxa de aprendizado ou camadas congeladas diferentes entre os experimentos. Só dá para afirmar a causa com configurações controladas.
- Como a acurácia global é 80%, ainda é possível que as classes raras (`AFIB`, `SA`) estejam mal classificadas, como se viu no Teste 2 da VGG16. Sem o relatório por classe, não dá para saber.

## Comparação com a VGG16

Métricas na melhor época (selecionada pela menor loss de validação):

| Métrica | ResNet50 `A_sem_aug` | VGG16 Teste 1 `A_sem_aug` | VGG16 Teste 2 `B_leve` | VGG16 Teste 3 `C_moderado` | VGG16 Teste 4 `D_pesado` |
|---|---:|---:|---:|---:|---:|
| Acurácia de validação (melhor época) | **80,32%** | 73,70% | 71,78% | 73,22% | 71,99% |
| Acurácia de validação (média das épocas) | **80,32%** | 71,04% | 71,14% | 70,01% | 69,02% |
| Loss de validação | **0,5471** | 0,7592 | 0,7840 | 0,8054 | 0,8382 |
| Acurácia de treino | **80,69%** | 67,24% | 65,97% | 63,96% | 60,40% |
| Loss de treino | **0,5391** | 0,9214 | 0,9563 | 1,0118 | 1,0927 |
| Melhor época / épocas executadas | 10 / 20 | 4 / 14 | 13 / 23 | 17 / 27 | 4 / 14 |
| Tempo total | 63,97 min | **49,58 min** | 90,32 min | 106,65 min | 63,58 min |
| Tempo médio por época | **3,20 min** | 3,54 min | 3,93 min | 3,95 min | 4,54 min |

**O que a tabela mostra:**

- **Frente ao baseline da VGG16 (mesmo `A_sem_aug`)**, a ResNet50 teve +6,61 p.p. de acurácia de validação (80,32% contra 73,70%) e loss de validação 0,2121 menor (0,5471 contra 0,7592).
- **A diferença é maior que a oscilação entre épocas.** Na VGG16, a oscilação da validação foi de 4,8 a 7,8 p.p. entre os testes. Aqui, até a pior época da ResNet50 (76,84%, época 1) supera a melhor época de qualquer teste VGG16 (máximo de 73,70%). Na média das épocas, a diferença é de +9,3 p.p. sobre o baseline VGG16.
- **O tempo total foi 14,39 min maior** (+29%) que o da VGG16 Teste 1, mas porque rodou 20 épocas em vez de 14. Por época, a ResNet50 foi mais rápida (3,20 contra 3,54 min), embora o ambiente de execução possa ter sido diferente.

**Ressalvas:** cada configuração foi executada uma única vez, e a comparação entre arquiteturas só é rigorosa se dataset, divisão dos dados, hiperparâmetros, pesos iniciais e seed forem os mesmos. Além disso, só há métricas de treino e validação; a comparação por classe e em conjunto de teste independente ainda não pode ser feita.

## Limitações

- Apenas **uma execução**; não há estimativa de variância (seeds diferentes).
- As métricas são de **treino e validação**. Não há resultado em conjunto de teste independente.
- Só há acurácia global. Faltam **relatório por classe** e **matriz de confusão**. O Teste 2 da VGG16 mostrou que a acurácia global pode esconder classes não reconhecidas (`AFIB`, `SA`).
- A "melhor época" é selecionada com a própria validação.
- Hiperparâmetros, pesos iniciais, dataset e divisão dos dados não estão documentados, o que limita a comparação com a VGG16.
- Testado apenas sem augmentation.

## Próximos passos

- [ ] Gerar o **relatório de classificação** (validação e teste) e a **matriz de confusão** do melhor modelo (época 10), com `zero_division=0`.
- [ ] Avaliar em um conjunto de **teste independente** e comparar por **F1 macro** com a VGG16.
- [ ] Confirmar que os hiperparâmetros, a divisão dos dados e os pesos iniciais são iguais aos da série VGG16.
- [ ] Testar a ResNet50 com `B_leve`, `C_moderado` e `D_pesado` para ver se o efeito do augmentation se repete.
- [ ] Testar **pesos de classe**, `WeightedRandomSampler` ou *focal loss* para o desbalanceamento.
- [ ] Considerar **F1 macro** como critério de seleção do modelo.
- [ ] Repetir com 3 a 5 seeds e reportar média ± desvio-padrão.
- [ ] Preencher a tabela de reprodutibilidade.

## Conclusão

A ResNet50 **sem data augmentation** atingiu **80,32% de acurácia de validação** e **loss de validação de 0,5471** na época 10, em 63,97 minutos (20 épocas). O treino evoluiu de forma contínua e a validação foi estável, sem os picos isolados vistos na VGG16. O resultado superou em cerca de 6,6 p.p. o baseline da VGG16, uma diferença maior que a oscilação entre épocas. Falta confirmar, com relatório por classe e teste independente, se esse ganho se estende às classes raras (`AFIB`, `SA`), e garantir que as duas arquiteturas foram treinadas sob as mesmas condições.