# Teste 3 — VGG16 com Data Augmentation Moderado

> Experimento da série de testes com VGG16 para classificação de ritmos cardíacos a partir de sinais de ECG representados como imagens, usando a estratégia de augmentation `C_moderado`.

## Sumário

1. [Objetivo](#objetivo)
2. [Configuração do experimento](#configuração-do-experimento)
3. [Resultados da melhor época](#resultados-da-melhor-época)
4. [Desempenho por classe](#desempenho-por-classe)
5. [Histórico de treinamento](#histórico-de-treinamento)
6. [Análise](#análise)
7. [Comparação entre os quatro testes](#comparação-entre-os-quatro-testes)
8. [Limitações](#limitações)
9. [Próximos passos](#próximos-passos)
10. [Conclusão](#conclusão)

## Objetivo

Avaliar o desempenho da arquitetura **VGG16** na classificação de sinais de ECG aplicando **data augmentation moderado** (`C_moderado`), nível intermediário entre o augmentation leve (Teste 2, `B_leve`) e o pesado (Teste 4, `D_pesado`). Com este teste, a série cobre quatro níveis de intensidade: nenhum, leve, moderado e pesado.

## Configuração do experimento

| Parâmetro | Valor |
|---|---|
| Arquitetura | VGG16 |
| Estratégia de augmentation | `C_moderado` — augmentation moderado |
| Número de classes | 6 |
| Épocas executadas | 27 |
| Melhor época | 17 |
| Critério de seleção | Menor loss de validação |
| Early stopping | Paciência de 10 épocas (consistente com melhor época 17 + 10 = 27 épocas executadas) |
| Tempo total de treinamento | 106,65 min (≈ 3,95 min/época) |

**Classes:** `AF`, `AFIB`, `SA`, `SB`, `SR`, `ST`

### Transformações do augmentation `C_moderado`

| Transformação | Parâmetros |
|---|---|
| _a preencher_ | _a preencher_ |
| _a preencher_ | _a preencher_ |

### Informações a completar

| Item | Valor |
|---|---|
| Dataset e significado das siglas das classes | _a preencher_ |
| Tamanho dos conjuntos de treino, validação e teste | _a preencher_ |
| Tamanho da imagem de entrada | _a preencher_ |
| Pesos iniciais (ImageNet ou aleatórios) | _a preencher_ |
| Otimizador e *learning rate* | _a preencher_ |
| Batch size | _a preencher_ |
| Tratamento de desbalanceamento (pesos de classe, amostragem) | _a preencher_ |
| Seed | _a preencher_ |
| GPU / ambiente | _a preencher_ |

## Resultados da melhor época

Melhor época: **17**, com a menor loss de validação do experimento. Neste teste, a época 17 também é a de **maior acurácia de validação** (73,22%), ou seja, os dois critérios apontam para o mesmo modelo.

| Métrica | Treino | Validação |
|---|---:|---:|
| Acurácia | 63,96% | **73,22%** |
| Loss | 1,0118 | **0,8054** |

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
| 1 | 1,0351 | 0,8286 | 62,15% | 72,34% |
| 2 | 1,0145 | 0,8588 | 63,08% | 66,79% |
| 3 | 1,0081 | 0,8364 | 63,59% | 72,67% |
| 4 | 1,0113 | 0,8285 | 63,76% | 70,70% |
| 5 | 1,0163 | 0,8426 | 63,49% | 69,85% |
| 6 | 1,0136 | 0,8338 | 63,83% | 69,71% |
| 7 | 1,0091 | 0,8876 | 64,05% | 65,99% |
| 8 | 1,0138 | 0,8273 | 63,85% | 71,03% |
| 9 | 1,0155 | 0,8159 | 63,50% | 72,38% |
| 10 | 1,0140 | 0,8269 | 63,73% | 71,61% |
| 11 | 1,0220 | 0,8594 | 63,44% | 67,58% |
| 12 | 1,0184 | 0,8306 | 63,40% | 68,97% |
| 13 | 1,0088 | 0,8192 | 63,81% | 69,85% |
| 14 | 1,0044 | 0,8148 | 63,77% | 71,09% |
| 15 | 1,0154 | 0,8210 | 63,47% | 70,60% |
| 16 | 1,0253 | 0,8380 | 63,35% | 68,43% |
| **17** | 1,0118 | **0,8054** | 63,96% | **73,22%** |
| 18 | 1,0136 | 0,8057 | 63,49% | 72,16% |
| 19 | 1,0071 | 0,8523 | 63,84% | 67,41% |
| 20 | 1,0190 | 0,8294 | 63,71% | 71,41% |
| 21 | 1,0167 | 0,8150 | 63,27% | 69,17% |
| 22 | 1,0189 | 0,8119 | 63,91% | 70,18% |
| 23 | 1,0085 | 0,8184 | 64,00% | 71,71% |
| 24 | 1,0112 | 0,8327 | 63,96% | 70,73% |
| 25 | 1,0125 | 0,8594 | 63,65% | 68,08% |
| 26 | 1,0161 | 0,8720 | 63,54% | 65,44% |
| 27 | 1,0147 | 0,8213 | 63,89% | 71,28% |

### Resumo estatístico (27 épocas)

| Métrica | Mínimo | Máximo | Média |
|---|---:|---:|---:|
| Acurácia de validação | 65,44% (ép. 26) | 73,22% (ép. 17) | 70,01% |
| Acurácia de treino | 62,15% (ép. 1) | 64,05% (ép. 7) | ≈ 63,6% |
| Loss de validação | 0,8054 (ép. 17) | 0,8876 (ép. 7) | — |
| Loss de treino | 1,0044 (ép. 14) | 1,0351 (ép. 1) | — |

## Análise

**Pontos observados nos dados:**

- O early stopping encerrou o treino na época 27, 10 épocas depois do melhor resultado (época 17). A loss de validação melhorou lentamente ao longo do treino (0,8286 na época 1 para 0,8054 na época 17). A época 18 ficou praticamente empatada com a melhor (0,8057).
- A **acurácia de treino estagna em ≈ 63–64%** desde a época 3, com loss de treino em torno de 1,01. O treino quase não evolui.
- A **acurácia de validação oscila bastante**: de 65,44% a 73,22% (amplitude de ≈ 7,8 p.p., a maior entre os testes com dados completos). A média das 27 épocas é 70,01%.
- **A melhor época é um pico entre valores instáveis.** Os 73,22% da época 17 ficam cerca de 3 p.p. acima da média das épocas. Por isso, a acurácia da "melhor época" tende a ser mais otimista que o desempenho típico do modelo.
- A validação fica acima do treino em **todas** as 27 épocas, comportamento compatível com augmentation aplicado só no treino (hipótese).

**Hipóteses a verificar (não confirmadas pelos dados):**

- Como nos testes 1, 2 e 4, o treino estagna cedo. O gargalo pode estar no *setup* de treinamento (learning rate, camadas congeladas, função de perda) e não na intensidade do augmentation.
- Nos testes 2 e 3 o modelo pode estar ignorando as classes raras, como se viu no Teste 2 (`AFIB` nunca previsto). Sem o relatório por classe, não é possível confirmar para este teste.

## Comparação entre os quatro testes

Métricas na melhor época (selecionada pela menor loss de validação):

| Métrica | Teste 1 `A_sem_aug` | Teste 2 `B_leve` | Teste 3 `C_moderado` | Teste 4 `D_pesado` |
|---|---:|---:|---:|---:|
| Acurácia de validação (melhor época) | **73,70%** | 71,78% | 73,22% | 71,99% |
| Acurácia de validação (média das épocas) | 71,04% | **71,14%** | 70,01% | 69,02% |
| Loss de validação | **0,7592** | 0,7840 | 0,8054 | 0,8382 |
| Acurácia de treino | **67,24%** | 65,97% | 63,96% | 60,40% |
| Loss de treino | **0,9214** | 0,9563 | 1,0118 | 1,0927 |
| Diferença validação − treino (acurácia) | 6,46 p.p. | 5,81 p.p. | 9,26 p.p. | 11,59 p.p. |
| Melhor época | 4 | 13 | 17 | 4 |
| Épocas executadas | 14 | 23 | 27 | 14 |
| Tempo total | **49,58 min** | 90,32 min | 106,65 min | 63,58 min |
| Tempo médio por época | 3,54 min | 3,93 min | 3,95 min | 4,54 min |
| Acurácia de teste | — | 72,26% | — | — |
| F1 macro (teste) | — | 0,4822 | — | — |

**O que a tabela mostra:**

- **Loss e métricas de treino pioram de forma consistente com a intensidade do augmentation.** A loss de treino sobe (0,92 → 0,96 → 1,01 → 1,09), a acurácia de treino cai (67,2% → 66,0% → 64,0% → 60,4%) e a loss de validação acompanha (0,759 → 0,784 → 0,805 → 0,838).
- **A acurácia de validação na melhor época não segue essa ordem** (73,70%, 71,78%, 73,22%, 71,99%). Como ela é um pico entre valores oscilantes, é pouco confiável para ranquear. A **média das épocas** é mais estável e mostra os testes 1 e 2 à frente (≈ 71%), seguidos do 3 (70,0%) e do 4 (69,0%).
- O tempo total depende sobretudo de quantas épocas o early stopping deixou rodar. O tempo por época não cresce de forma regular com o augmentation (o Teste 4 foi o mais lento por época), então não dá para atribuir o tempo à intensidade das transformações.
- Os melhores resultados de acurácia por época de validação (testes 1 e 3) diferem em apenas 0,48 p.p., muito menos que a oscilação entre épocas (até ≈ 7,8 p.p.).

**Ressalva:** são execuções únicas, e as diferenças em acurácia são menores que a oscilação entre épocas. Não devem ser tratadas como estatisticamente comprovadas. A comparação por classe (F1 macro) só existe, até agora, para o Teste 2.

## Limitações

- Apenas **uma execução** por configuração; não há estimativa de variância.
- Faltam **acurácia e loss de teste**, **relatório por classe** e **matriz de confusão** para este teste.
- A seleção da melhor época usa a loss de validação, e a acurácia dessa época é um pico entre valores muito oscilantes.
- As transformações do `C_moderado` e os hiperparâmetros não estão documentados.

## Próximos passos

- [ ] Gerar o **relatório de classificação** (validação e teste) e a **matriz de confusão** do melhor modelo (época 17).
- [ ] Gerar os mesmos relatórios para os **testes 1 e 4** e fechar a comparação por **F1 macro** entre os quatro testes.
- [ ] Comparar também a **média/mediana das últimas épocas**, e não só a melhor época.
- [ ] Testar **pesos de classe**, `WeightedRandomSampler` ou *focal loss* para tratar o desbalanceamento.
- [ ] Investigar a estagnação do treino (learning rate, camadas congeladas, scheduler).
- [ ] Repetir com 3 a 5 seeds e reportar média ± desvio-padrão.
- [ ] Documentar as transformações do `C_moderado` e preencher a tabela de reprodutibilidade.

## Conclusão

A VGG16 com **augmentation moderado (`C_moderado`)** atingiu **73,22% de acurácia de validação** e **loss de validação de 0,8054** na época 17, em 106,65 minutos (27 épocas). Entre as três estratégias com augmentation, foi a de maior acurácia de validação na melhor época, mas a média das épocas (70,01%) e a loss de validação ficam atrás do baseline sem augmentation. Comparando os quatro testes, quanto maior o augmentation, maiores as losses e menor a acurácia de treino, sem ganho claro de generalização. A conclusão definitiva depende de avaliação em teste independente, métricas por classe e repetições do experimento.