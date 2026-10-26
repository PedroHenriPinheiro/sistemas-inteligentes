# Teste 2 — VGG16 com Data Augmentation Leve

> Experimento da série de testes com VGG16 para classificação de ritmos cardíacos a partir de sinais de ECG representados como imagens, usando a estratégia de augmentation `B_leve`.

## Sumário

1. [Objetivo](#objetivo)
2. [Configuração do experimento](#configuração-do-experimento)
3. [Resultados da melhor época](#resultados-da-melhor-época)
4. [Desempenho por classe](#desempenho-por-classe)
5. [Histórico de treinamento](#histórico-de-treinamento)
6. [Análise](#análise)
7. [Comparação entre os testes](#comparação-entre-os-testes)
8. [Limitações](#limitações)
9. [Próximos passos](#próximos-passos)
10. [Conclusão](#conclusão)

## Objetivo

Avaliar o desempenho da arquitetura **VGG16** na classificação de sinais de ECG aplicando **data augmentation leve** (`B_leve`), um nível intermediário entre o baseline sem augmentation (Teste 1) e o augmentation pesado (Teste 4).

## Configuração do experimento

| Parâmetro | Valor |
|---|---|
| Arquitetura | VGG16 |
| Estratégia de augmentation | `B_leve` — augmentation leve |
| Número de classes | 6 |
| Épocas máximas | 100 |
| Épocas executadas | 23 |
| Melhor época | 13 |
| Critério de seleção | Menor loss de validação |
| Early stopping | Ativado, paciência de 10 épocas sem melhora |
| Tempo total de treinamento | 90,32 min (≈ 3,9 min/época) |

**Classes:** `AF`, `AFIB`, `SA`, `SB`, `SR`, `ST`

**Tamanho dos conjuntos avaliados:** validação com 6.351 amostras e teste com 6.342 amostras.

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
| Dataset e significado das siglas das classes | _a preencher_ |
| Tamanho do conjunto de treino | _a preencher_ |
| Pesos iniciais (ImageNet ou aleatórios) | _a preencher_ |
| Otimizador e *learning rate* | _a preencher_ |
| Batch size | _a preencher_ |
| Tratamento de desbalanceamento (pesos de classe, amostragem) | _a preencher_ |
| Seed | _a preencher_ |
| GPU / ambiente | _a preencher_ |

## Resultados da melhor época

Melhor época: **13**, com a menor loss de validação do experimento. A acurácia de validação da época 13 (71,78%) coincide com a acurácia do relatório de classificação na validação, o que confirma que o relatório foi gerado com o melhor modelo.

| Métrica | Treino | Validação | Teste |
|---|---:|---:|---:|
| Acurácia | 65,97% | **71,78%** | **72,26%** |
| Loss | 0,9563 | **0,7840** | — |
| Macro F1 | — | 0,4812 | 0,4822 |
| F1 ponderado | — | 0,6745 | 0,6801 |

A acurácia de teste (72,26%) é muito próxima da de validação (71,78%), o que indica que a seleção do melhor modelo pela validação não superestimou o desempenho de forma relevante.

### Curva de acurácia

![Acurácia - VGG16 (B_leve)](acuracia_vgg16_b_leve.png)

> Salve o gráfico enviado com o nome `acuracia_vgg16_b_leve.png` na mesma pasta deste README para a imagem aparecer.

## Desempenho por classe

### Conjunto de teste (6.342 amostras)

| Classe | Precisão | Revocação | F1 | Suporte | % do teste |
|---|---:|---:|---:|---:|---:|
| AF | 0,6804 | 0,4012 | 0,5047 | 1.194 | 18,83% |
| AFIB | 0,0000 | 0,0000 | 0,0000 | 267 | 4,21% |
| SA | 0,7368 | 0,0515 | 0,0962 | 272 | 4,29% |
| SB | 0,8283 | 0,9860 | 0,9003 | 2.363 | 37,26% |
| SR | 0,6107 | 0,6785 | 0,6428 | 1.179 | 18,59% |
| ST | 0,6417 | 0,8997 | 0,7491 | 1.067 | 16,82% |
| **Acurácia** | | | **0,7226** | 6.342 | |
| **Média macro** | 0,5830 | 0,5028 | 0,4822 | 6.342 | |
| **Média ponderada** | 0,6898 | 0,7226 | 0,6801 | 6.342 | |

### Conjunto de validação (6.351 amostras)

| Classe | Precisão | Revocação | F1 | Suporte |
|---|---:|---:|---:|---:|
| AF | 0,6686 | 0,3833 | 0,4872 | 1.195 |
| AFIB | 0,0000 | 0,0000 | 0,0000 | 267 |
| SA | 0,9000 | 0,0667 | 0,1241 | 270 |
| SB | 0,8164 | 0,9899 | 0,8948 | 2.372 |
| SR | 0,6114 | 0,6610 | 0,6353 | 1.183 |
| ST | 0,6392 | 0,8957 | 0,7460 | 1.064 |
| **Acurácia** | | | **0,7178** | 6.351 |
| **Média macro** | 0,6059 | 0,4994 | 0,4812 | 6.351 |
| **Média ponderada** | 0,6900 | 0,7178 | 0,6745 | 6.351 |

### Leitura por classe (teste)

- **SB** é a classe mais bem resolvida: revocação de 98,60% e F1 de 0,90. É também a classe majoritária (37,26% do conjunto).
- **ST** tem revocação alta (89,97%) com precisão de 64,17%, ou seja, o modelo atribui `ST` a muitas amostras de outras classes.
- **SR** tem desempenho intermediário e equilibrado (precisão 61,07%, revocação 67,85%).
- **AF** acerta pouco mais de 40% dos casos reais (revocação 40,12%).
- **SA** quase nunca é reconhecida: revocação de 5,15%. As poucas previsões de `SA` têm precisão de 73,68%.
- **AFIB** tem precisão, revocação e F1 iguais a zero. O aviso `UndefinedMetricWarning` do scikit-learn indica que o modelo **não previu AFIB em nenhuma amostra**, então as 267 amostras reais foram classificadas como outras classes.

## Histórico de treinamento

| Época | Loss treino | Loss val. | Acc. treino | Acc. val. |
|---:|---:|---:|---:|---:|
| 1 | 0,9821 | 0,8132 | 64,17% | 72,74% |
| 2 | 0,9604 | 0,8217 | 65,33% | 68,98% |
| 3 | 0,9612 | 0,7976 | 65,50% | 73,17% |
| 4 | 0,9605 | 0,7869 | 65,67% | 72,63% |
| 5 | 0,9639 | 0,8400 | 65,88% | 68,57% |
| 6 | 0,9631 | 0,8157 | 66,02% | 72,00% |
| 7 | 0,9596 | 0,8346 | 65,92% | 69,37% |
| 8 | 0,9581 | 0,7906 | 65,44% | 72,67% |
| 9 | 0,9693 | 0,7933 | 65,30% | 70,41% |
| 10 | 0,9653 | 0,8065 | 65,56% | 71,69% |
| 11 | 0,9703 | 0,8356 | 65,47% | 68,04% |
| 12 | 0,9648 | 0,8139 | 65,52% | 69,17% |
| **13** | 0,9563 | **0,7840** | 65,97% | 71,78% |
| 14 | 0,9630 | 0,7963 | 65,82% | 71,17% |
| 15 | 0,9669 | 0,8012 | 65,66% | 72,70% |
| 16 | 0,9676 | 0,7936 | 65,79% | 71,85% |
| 17 | 0,9615 | 0,7936 | 65,82% | 72,84% |
| 18 | 0,9636 | 0,7882 | 65,67% | 73,06% |
| 19 | 0,9582 | 0,8454 | 66,04% | 67,20% |
| 20 | 0,9660 | 0,8133 | 65,84% | 73,17% |
| 21 | 0,9637 | 0,8219 | 65,60% | 68,46% |
| 22 | 0,9761 | 0,7941 | 65,33% | 71,97% |
| 23 | 0,9567 | 0,7960 | 65,88% | 72,56% |

### Resumo estatístico (23 épocas)

| Métrica | Mínimo | Máximo | Média |
|---|---:|---:|---:|
| Acurácia de validação | 67,20% (ép. 19) | 73,17% (ép. 3 e 20) | 71,14% |
| Acurácia de treino | 64,17% (ép. 1) | 66,04% (ép. 19) | ≈ 65,6% |
| Loss de validação | 0,7840 (ép. 13) | 0,8454 (ép. 19) | — |
| Loss de treino | 0,9563 (ép. 13) | 0,9821 (ép. 1) | — |

## Análise

**Pontos observados nos dados:**

- O early stopping encerrou o treino na época 23, 10 épocas depois do melhor resultado (época 13). A melhor loss de validação passou por três marcos (épocas 1, 3–4 e 13), com melhoras pequenas: de 0,7869 na época 4 para 0,7840 na época 13.
- **A melhor época pela loss não é a de maior acurácia.** A maior acurácia de validação (73,17%) ocorreu nas épocas 3 e 20, enquanto a época escolhida (13) tem 71,78%. Os dois critérios podem apontar modelos diferentes.
- A **acurácia de treino é quase constante** (64,2% a 66,0%) e a loss de treino fica em torno de 0,96 desde a época 2. O treino praticamente não evolui.
- A **acurácia de validação oscila muito**: de 67,20% a 73,17%, amplitude de cerca de 6 p.p., sem tendência de melhora. Ela segue sempre acima da de treino. O augmentation é aplicado só no treino (confirmado na configuração), o que pode contribuir, mas o mesmo ocorre no Teste 1, sem augmentation, então não deve ser a única explicação.
- **A acurácia global esconde o problema principal.** A acurácia de teste é 72,26%, mas o F1 macro é 0,4822. O desempenho depende fortemente da classe.

**Hipóteses a verificar (não confirmadas pelos dados):**

- **Desbalanceamento de classes.** As duas classes com pior desempenho (`AFIB` com 4,21% e `SA` com 4,29% do teste) são também as mais raras. A classe `SB` (37,26%) é a mais bem classificada. Sem pesos de classe ou reamostragem, o modelo tende a favorecer as majoritárias.
- **Confusão entre classes parecidas.** Só a matriz de confusão mostra para onde as amostras de `AFIB` e `SA` estão indo. Não é possível afirmar a partir do relatório.
- **Estagnação do treino.** Como o treino já estagnava no Teste 1, o limite parece estar no *setup* de treinamento (learning rate, camadas congeladas, função de perda), e não na quantidade de augmentation.

## Comparação entre os testes

Métricas na melhor época (selecionada pela menor loss de validação):

| Métrica | Teste 1 (`A_sem_aug`) | Teste 2 (`B_leve`) | Teste 4 (`D_pesado`) |
|---|---:|---:|---:|
| Acurácia de validação | **73,70%** | 71,78% | 71,99% |
| Loss de validação | **0,7592** | 0,7840 | 0,8382 |
| Acurácia de treino | 67,24% | 65,97% | 60,40% |
| Loss de treino | 0,9214 | 0,9563 | 1,0927 |
| Melhor época | 4 | 13 | 4 |
| Épocas executadas | 14 | 23 | 14 |
| Tempo total | **49,58 min** | 90,32 min | 63,58 min |
| Tempo médio por época | 3,54 min | 3,93 min | 4,54 min |

**O que a tabela mostra:**

- Da esquerda para a direita (mais augmentation), a **loss de treino sobe** (0,92 → 0,96 → 1,09) e a **acurácia de treino cai** (67,2% → 66,0% → 60,4%). A loss de validação segue a mesma ordem (0,759 → 0,784 → 0,838).
- A acurácia de validação não segue essa ordem: os testes 2 e 4 ficaram praticamente empatados (71,78% vs 71,99%).
- O Teste 2 levou mais tempo total porque rodou mais épocas (23 vs 14), não porque cada época seja mais lenta. O tempo por época ficou entre os dois outros testes.

**Ressalva:** são execuções únicas e a oscilação da acurácia de validação entre épocas (4 a 6 p.p.) é maior que as diferenças entre os testes. Essas diferenças não devem ser tratadas como estatisticamente comprovadas. Além disso, o relatório por classe existe só para este teste até agora; sem ele nos outros, a comparação por F1 macro ainda não pode ser feita.

## Limitações

- Apenas **uma execução** por configuração; não há estimativa de variância.
- Faltam **matriz de confusão** e **loss de teste**.
- A seleção da melhor época usa a loss de validação; escolher pela acurácia ou pelo F1 macro levaria a outra época.
- Os hiperparâmetros não estão documentados, o que limita a interpretação.
- Não está documentado se houve tratamento de desbalanceamento.

## Próximos passos

- [ ] Gerar a **matriz de confusão** (teste) para ver para onde vão `AFIB` e `SA`.
- [ ] Gerar os mesmos relatórios por classe para os **testes 1 e 4** e comparar por **F1 macro**.
- [ ] Testar **pesos de classe**, `WeightedRandomSampler` ou *focal loss* para tratar o desbalanceamento.
- [ ] Usar `zero_division=0` em `classification_report` para eliminar os avisos.
- [ ] Considerar **F1 macro** como critério de seleção do modelo ou de early stopping.
- [ ] Salvar também a curva de loss (treino e validação) junto com a de acurácia.
- [ ] Repetir com 3 a 5 seeds e reportar média ± desvio-padrão.
- [ ] Preencher a tabela de reprodutibilidade.

## Conclusão

A VGG16 com **augmentation leve (`B_leve`)** atingiu **71,78% de acurácia de validação** e **72,26% de acurácia de teste**, com loss de validação de 0,7840 na época 13, em 90,32 minutos. O resultado fica entre o baseline sem augmentation e o augmentation pesado nas métricas de loss, sem ganho claro em acurácia. O ponto mais importante do experimento é o **F1 macro de 0,48**: o modelo reconhece bem `SB`, `ST` e razoavelmente `SR`, mas **não identifica `AFIB`** e quase não identifica `SA`. O próximo avanço provavelmente depende de tratar o desbalanceamento e o setup de treinamento, mais do que da intensidade do augmentation.