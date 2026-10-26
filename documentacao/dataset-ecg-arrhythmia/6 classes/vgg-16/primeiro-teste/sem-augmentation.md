# Teste 1 — VGG16 sem Data Augmentation

> Experimento de referência (*baseline*) da série de testes com VGG16 para classificação de ritmos cardíacos a partir de sinais de ECG representados como imagens.

## Sumário

1. [Objetivo](#objetivo)
2. [Configuração do experimento](#configuração-do-experimento)
3. [Resultados da melhor época](#resultados-da-melhor-época)
4. [Histórico de treinamento](#histórico-de-treinamento)
5. [Análise](#análise)
6. [Comparação com o Teste 4](#comparação-com-o-teste-4)
7. [Limitações](#limitações)
8. [Próximos passos](#próximos-passos)
9. [Conclusão](#conclusão)

## Objetivo

Avaliar o desempenho da arquitetura **VGG16** na classificação de sinais de ECG sem aplicar nenhuma técnica de *data augmentation*. Este teste serve como **referência** para medir o efeito das demais estratégias de augmentation (como o `D_pesado`, do Teste 4).

## Configuração do experimento

| Parâmetro | Valor |
|---|---|
| Arquitetura | VGG16 |
| Estratégia de augmentation | `A_sem_aug` — nenhuma |
| Número de classes | 6 |
| Épocas executadas | 14 |
| Melhor época | 4 |
| Tempo total de execução | 49,58 min (≈ 3,5 min/época) |

**Classes:** `AF`, `AFIB`, `SA`, `SB`, `SR`, `ST`

### Pré-processamento

Sem augmentation, o treino usa o mesmo pipeline de validação e teste (`transform_avaliacao`), comum a todos os testes da série:

| Transformação | Parâmetros |
|---|---|
| `Resize` | 224 × 224 |
| `ToTensor` + `Normalize` | média `[0.485, 0.456, 0.406]`, desvio `[0.229, 0.224, 0.225]` |

### Informações a completar

Preencha para tornar o experimento reprodutível:

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
| Acurácia | 67,24% | **73,70%** |
| Loss | 0,9214 | **0,7592** |

## Histórico de treinamento

| Época | Loss treino | Loss val. | Acc. treino | Acc. val. |
|---:|---:|---:|---:|---:|
| 1 | 0,9442 | 0,7932 | 66,12% | 72,62% |
| 2 | 0,9212 | 0,7948 | 67,10% | 69,55% |
| 3 | 0,9197 | 0,7821 | 67,03% | 73,33% |
| **4** | 0,9214 | **0,7592** | 67,24% | **73,70%** |
| 5 | 0,9225 | 0,7994 | 67,46% | 70,54% |
| 6 | 0,9267 | 0,7898 | 67,10% | 70,68% |
| 7 | 0,9233 | 0,8214 | 67,32% | 69,03% |
| 8 | 0,9206 | 0,7926 | 67,51% | 71,30% |
| 9 | 0,9221 | 0,7608 | 67,17% | 71,41% |
| 10 | 0,9230 | 0,7854 | 67,31% | 70,93% |
| 11 | 0,9267 | 0,8051 | 67,22% | 70,11% |
| 12 | 0,9202 | 0,8118 | 67,39% | 68,92% |
| 13 | 0,9172 | 0,7718 | 67,37% | 71,64% |
| 14 | 0,9222 | 0,7776 | 67,23% | 70,78% |

### Resumo estatístico (14 épocas)

| Métrica | Mínimo | Máximo | Média |
|---|---:|---:|---:|
| Acurácia de validação | 68,92% (ép. 12) | 73,70% (ép. 4) | 71,04% |
| Acurácia de treino | 66,12% (ép. 1) | 67,51% (ép. 8) | ≈ 67,2% |
| Loss de validação | 0,7592 (ép. 4) | 0,8214 (ép. 7) | — |
| Loss de treino | 0,9172 (ép. 13) | 0,9442 (ép. 1) | — |

## Análise

**Pontos observados nos dados:**

- O melhor resultado aparece cedo, na **época 4**, e nenhuma das 10 épocas seguintes o superou. A execução total de 14 épocas (4 + 10) é compatível com um *early stopping* com paciência de 10, mas confirme isso na sua configuração.
- A **loss e a acurácia de treino estagnam logo após a época 1–2**: a acurácia fica entre 66% e 67,5% e a loss em torno de 0,92. Ou seja, o modelo praticamente parou de aprender a partir daí.
- A **acurácia de validação oscila bastante** (68,9% a 73,7%, amplitude de ≈ 4,8 p.p.) sem tendência de melhora.
- A acurácia de **validação é maior que a de treino** em todas as épocas, o que não é o comportamento clássico de sobreajuste.

**Hipóteses a verificar (não confirmadas pelos dados):**

- Treino e validação podem ser medidos em condições diferentes (por exemplo, *dropout* ativo apenas no treino). Isso costuma explicar acurácia de treino menor.
- A divisão pode gerar um conjunto de validação mais "fácil" ou com distribuição de classes diferente do treino.
- A estagnação precoce sugere que o treinamento está limitado por algo como *learning rate*, congelamento de camadas ou capacidade de representação, e não por falta de dados. Vale inspecionar esses pontos.

## Comparação com o Teste 4

| Métrica | Teste 1 (`A_sem_aug`) | Teste 4 (`D_pesado`) | Diferença |
|---|---:|---:|---:|
| Acurácia de validação (melhor) | **73,70%** | 71,99% | +1,71 p.p. |
| Loss de validação (melhor) | **0,7592** | 0,8382 | −0,0790 |
| Acurácia de treino (melhor época) | 67,24% | 60,40% | +6,84 p.p. |
| Loss de treino (melhor época) | 0,9214 | 1,0927 | −0,1713 |
| Tempo de execução | **49,58 min** | 63,58 min | −14,00 min |

Nestes dois experimentos, a configuração sem augmentation obteve métricas de validação melhores e foi mais rápida. Como a diferença de 1,71 p.p. é **menor que a oscilação entre épocas** (até ≈ 4,8 p.p. neste teste) e cada teste foi rodado uma única vez, ela **não deve ser tratada como diferença estatisticamente comprovada**.

## Limitações

- Apenas **uma execução** por configuração; sem repetições com seeds diferentes não há estimativa de variância.
- As métricas são de **treino e validação**. Não há resultado em conjunto de teste independente.
- Só há acurácia global; falta análise **por classe**, importante se houver desbalanceamento entre as seis classes.
- A "melhor época" é selecionada com base na própria validação, o que pode superestimar o desempenho real.

## Próximos passos

- [ ] Avaliar o melhor modelo (época 4) em um conjunto de **teste independente**.
- [ ] Gerar **matriz de confusão** e **precisão, revocação e F1 por classe**.
- [ ] Registrar a distribuição de classes em treino e validação.
- [ ] Repetir o experimento com 3 a 5 seeds e reportar média ± desvio-padrão.
- [ ] Investigar a estagnação do treino (learning rate, camadas congeladas, scheduler).
- [ ] Preencher a tabela de informações de reprodutibilidade acima.

## Conclusão

A VGG16 treinada **sem data augmentation** atingiu **73,70% de acurácia de validação** e **loss de validação de 0,7592**, ambas na época 4, em 49,58 minutos. O treino estagnou cedo e a validação oscilou sem melhorar depois do pico. Este teste estabelece a **linha de base** para as comparações com as demais estratégias de augmentation, a serem consolidadas após a avaliação em teste independente.