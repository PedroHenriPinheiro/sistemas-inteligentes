# Acurácia geral da cascata (modelos 1, 2, 3 e 4)

Resultado final do experimento de **cascata de ResNet1D** para classificação de arritmias em ECG de 12 derivações (base PhysioNet, *A large scale 12-lead ECG database for arrhythmia study*, 9 classes de ritmo). Este documento reúne apenas a avaliação conjunta dos 4 modelos. Treino, arquitetura e análise de cada modelo estão no [README geral](README.md) e nos READMEs de cada pasta.

## Sumário

- [Resultado](#resultado)
- [Cascata avaliada](#cascata-avaliada)
- [Modelos e configurações usados](#modelos-e-configurações-usados)
- [Como a acurácia foi calculada](#como-a-acurácia-foi-calculada)
- [Resultado por classe original](#resultado-por-classe-original)
- [Resultado por grupo](#resultado-por-grupo)
- [De onde vêm os erros](#de-onde-vêm-os-erros)
- [Sensibilidade ao modelo 2](#sensibilidade-ao-modelo-2)
- [Conferência dos resultados](#conferência-dos-resultados)
- [Limitações](#limitações)
- [Próximos passos](#próximos-passos)

## Resultado

Conjunto de teste, 6491 exames, 9 classes originais.

| Métrica | Valor |
|---|---|
| **Acurácia geral da cascata (estimada)** | **90,53%** (cerca de 5876 acertos em 6491) |
| Acurácia balanceada nas 9 classes (estimada) | 0,7544 |
| Acurácia do modelo 1 isolado (4 grupos) | 98,01% |
| Limite superior, se o modelo 2 fosse perfeito | 94,47% |

A acurácia da cascata fica **7,5 pontos percentuais abaixo** da do modelo 1, porque os erros dos especialistas se somam aos do roteamento. O principal responsável é o modelo 2 (AFIB × AF), que não aprendeu a tarefa.

> **Importante:** este valor é uma **estimativa** feita a partir das matrizes de confusão de cada modelo, supondo que o erro do modelo 1 é independente da classe dentro do grupo. Não é uma medição ponta a ponta. Ver [Limitações](#limitações).

## Cascata avaliada

```
ECG (12 × 5000)
      │
      ▼
 Modelo 1 ── AFIB ──▶ Modelo 2: AFIB × AF
      │
      ├───── GSVT ──▶ Modelo 3: SVT, AT, AVRT, ST
      │
      ├───── SB
      │
      └───── SR ────▶ Modelo 4: SR × SA
```

O modelo 1 decide o grupo. Se o grupo estiver errado, o exame chega a um especialista que não contém a classe verdadeira (ou ao rótulo SB), e o resultado final está errado de qualquer forma.

## Modelos e configurações usados

| Modelo | Tarefa | Configuração | Acurácia (teste) | Exames de teste |
|---|---|---|---|---|
| 1 | AFIB, GSVT, SB, SR | earlystop15 | 98,01% | 6491 |
| 2 | AFIB × AF | descongelamento40 | 82,07% (equivale a sempre prever AF) | 1461 |
| 3 | SVT, AT, AVRT, ST | earlystop15 | 96,20% | 1209 |
| 4 | SR × SA | earlystop15 | 86,83% | 1450 |

O modelo 2 usa `descongelamento40` porque `earlystop15` não tem JSON nem matrizes de confusão, só curvas. As curvas sugerem que ele também seria o classificador trivial.

## Como a acurácia foi calculada

Para cada classe original `c`, no grupo `g`:

```
P(acerto | c) = recall do grupo g no modelo 1  ×  recall da classe c no especialista do grupo g
```

Para SB, que não tem especialista, vale só o recall do grupo SB no modelo 1.

A acurácia geral é a soma dos acertos esperados de todas as classes dividida por 6491:

```
acurácia = Σ_c  n_teste(c) × P(acerto | c)  /  6491
```

Matrizes de confusão de teste usadas:

**Modelo 1 (earlystop15)**

| Real \ Previsto | AFIB | GSVT | SB | SR |
|---|---|---|---|---|
| **AFIB** | 1427 | 31 | 1 | 2 |
| **GSVT** | 8 | 1178 | 3 | 20 |
| **SB** | 3 | 0 | 2352 | 16 |
| **SR** | 2 | 22 | 21 | 1405 |

Recall dos grupos: AFIB 0,9767 · GSVT 0,9744 · SB 0,9920 · SR 0,9690.

**Modelo 2 (descongelamento40)**

| Real \ Previsto | AFIB | AF |
|---|---|---|
| **AFIB** | 0 | 262 |
| **AF** | 0 | 1199 |

**Modelo 3 (earlystop15)**

| Real \ Previsto | SVT | AT | AVRT | ST |
|---|---|---|---|---|
| **SVT** | 80 | 1 | 1 | 2 |
| **AT** | 4 | 31 | 1 | 5 |
| **AVRT** | 1 | 0 | 3 | 0 |
| **ST** | 5 | 25 | 1 | 1049 |

**Modelo 4 (earlystop15)**

| Real \ Previsto | SR | SA |
|---|---|---|
| **SR** | 1101 | 85 |
| **SA** | 106 | 158 |

## Resultado por classe original

| Classe | Exames de teste | Recall do grupo (modelo 1) | Recall no especialista | Recall na cascata | Acertos esperados |
|---|---|---|---|---|---|
| AFIB | 262 | 0,9767 | 0,0000 | 0,0000 | 0,0 |
| AF | 1199 | 0,9767 | 1,0000 | 0,9767 | 1171,1 |
| SVT | 84 | 0,9744 | 0,9524 | 0,9280 | 77,9 |
| AT | 41 | 0,9744 | 0,7561 | 0,7367 | 30,2 |
| AVRT | 4 | 0,9744 | 0,7500 | 0,7308 | 2,9 |
| ST | 1080 | 0,9744 | 0,9713 | 0,9464 | 1022,1 |
| SB | 2371 | 0,9920 | (sem especialista) | 0,9920 | 2352,0 |
| SR | 1186 | 0,9690 | 0,9283 | 0,8995 | 1066,8 |
| SA | 264 | 0,9690 | 0,5985 | 0,5799 | 153,1 |
| **Total** | **6491** | | | | **≈ 5876,2 (90,53%)** |

## Resultado por grupo

| Grupo | Classes | Exames de teste | Acertos esperados | Acurácia na cascata |
|---|---|---|---|---|
| AFIB | AFIB, AF | 1461 | 1171 | 80,16% |
| GSVT | SVT, AT, AVRT, ST | 1209 | 1133 | 93,73% |
| SB | SB | 2371 | 2352 | 99,20% |
| SR | SR, SA | 1450 | 1220 | 84,13% |

## De onde vêm os erros

Dos cerca de 615 erros esperados em 6491 exames:

| Origem | Erros esperados |
|---|---|
| Roteamento do modelo 1 (grupo errado) | ≈ 129 |
| Especialistas (modelos 2, 3 e 4) | ≈ 486 |

Os maiores pontos de perda:

1. **AFIB × AF (modelo 2):** os 262 exames de AFIB são previstos como AF. Sozinho, isso custa 4,0 pontos percentuais da acurácia geral.
2. **SA (modelo 4):** 106 SA previstos como SR e 85 SR previstos como SA no teste (antes do efeito do roteamento).
3. **AT e AVRT (modelo 3):** poucos exames (41 e 4), mas com recall de 0,76 e 0,75 e 25 ST previstos como AT.

## Sensibilidade ao modelo 2

Acurácia geral da cascata para diferentes recalls de AFIB no modelo 2 (mantendo o recall de AF em 100% e o resto igual):

| Recall de AFIB no modelo 2 | Acurácia da cascata |
|---|---|
| 0,0 (situação atual) | 90,53% |
| 0,2 | 91,32% |
| 0,4 | 92,11% |
| 0,6 | 92,89% |
| 0,8 | 93,68% |
| 1,0 (limite superior) | 94,47% |

Cada 10 pontos percentuais de recall de AFIB no modelo 2 somam cerca de 0,4 ponto percentual à acurácia geral.

## Conferência dos resultados

Os números dos READMEs foram recalculados a partir das matrizes de confusão de teste (tabelas e imagens `matriz_teste.png`):

- Acurácia, acurácia balanceada, F1 macro e recalls dos modelos 1, 2, 3 e 4 batem com os valores documentados.
- A soma das linhas das matrizes fecha com o número de exames de teste de cada classe, e os 4 grupos somam 6491.
- Os pesos de classe seguem `w = sqrt(N / (K · n_classe))`, e as épocas executadas batem com melhor época + 15 (ou + 5).
- A tabela "Estimativa grosseira por classe original" do [README geral](README.md) é reproduzida exatamente.

## Limitações

- **Não é avaliação ponta a ponta.** Os modelos 2, 3 e 4 foram avaliados só com exames do grupo real. Na cascata de verdade, os exames que o modelo 1 encaminha ao grupo errado chegam ao especialista, e a distribuição desses exames pode ser diferente.
- **Suposição de independência.** Usar o recall do grupo para todas as classes dele pode ser otimista, por exemplo se o SA for mais difícil para o modelo 1 do que o SR. A matriz do modelo 1 é por grupo, não por classe original.
- **Modelo 2 sem `earlystop15` confirmado.** Só `descongelamento40` tem métricas, e ele colapsa para AF.
- **Classes muito pequenas:** AVRT tem 4 exames de teste e AT tem 41, então os recalls dessas classes têm intervalos de confiança largos.
- **Uma execução por configuração** (semente 42), sem média nem desvio entre execuções.
- **Sem avaliação em dados externos à base.**

## Próximos passos

1. **Script de inferência ponta a ponta:** rodar o modelo 1 no conjunto de teste, encaminhar cada exame ao especialista do grupo previsto e medir acurácia, F1 e a matriz de confusão nas 9 classes originais. Isso exige os pesos `.pt` e o `X_teste`.
2. **Corrigir o modelo 2:** gerar JSON e matrizes de `earlystop15`, acompanhar AUC e acurácia balanceada por época, ajustar o limiar de decisão ou usar amostragem balanceada. É onde há mais ganho possível (até 3,9 pontos percentuais).
3. **Melhorar o SA no modelo 4:** features de variabilidade da frequência cardíaca (HRV), ajuste de limiar e revisão dos rótulos.
4. **Repetir com outras sementes** e reportar intervalos de confiança para a acurácia da cascata.