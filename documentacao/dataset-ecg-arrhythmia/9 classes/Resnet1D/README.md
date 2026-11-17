# Classificação de arritmias em ECG de 12 derivações com CNN 1D (cascata de modelos): experimento com pesos de classe e scheduler

Classificação de arritmias direto no sinal de ECG (12 derivações × 5000 amostras, 500 Hz) usando uma **ResNet1D** treinada do zero, organizada como uma **cascata de modelos**: um modelo geral separa grupos de ritmos e modelos especialistas refinam os grupos que reúnem várias classes originais.

Este é o **segundo experimento**. Em relação ao [experimento anterior](#comparação-com-o-experimento-anterior) (perda sem pesos, LR fixo, paciência 5), acrescenta **pesos de classe pela raiz inversa da frequência** e **`ReduceLROnPlateau`**, e compara duas durações de treino: `descongelamento40` e `earlystop15`.

Base de dados: PhysioNet, *A large scale 12-lead ECG database for arrhythmia study*.

## Sumário

- [Visão geral da cascata](#visão-geral-da-cascata)
- [Resumo dos resultados](#resumo-dos-resultados)
- [Comparação com o experimento anterior](#comparação-com-o-experimento-anterior)
- [Dados](#dados)
- [Arquitetura](#arquitetura)
- [Configuração de treino](#configuração-de-treino)
- [Documentação por modelo](#documentação-por-modelo)
- [Como reproduzir](#como-reproduzir)
- [Estrutura do repositório](#estrutura-do-repositório)
- [Limitações e próximos passos](#limitações-e-próximos-passos)
- [Glossário](#glossário)

## Visão geral da cascata

A base original tem 9 classes de ritmo. Em vez de treinar um único classificador de 9 classes, o plano é usar 4 modelos:

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

| Modelo | Tarefa | Classes | Pasta |
|---|---|---|---|
| 1 | Classificador geral (grupos) | AFIB, GSVT, SB, SR | [`modelo1/`](modelo1/README.md) |
| 2 | Especialista do grupo AFIB | AFIB, AF | [`modelo2/`](modelo2/README.md) |
| 3 | Especialista do grupo GSVT | SVT, AT, AVRT, ST | [`modelo3/`](modelo3/README.md) |
| 4 | Especialista do grupo SR | SR, SA | [`modelo4/`](modelo4/README.md) |

Agrupamento usado no modelo 1 (definido no `classes_9c.json`, campo `grupo_4c`):

| Grupo | Classes originais |
|---|---|
| AFIB | AFIB, AF |
| GSVT | SVT, AT, AVRT, ST |
| SB | SB |
| SR | SR, SA |

> O SA pertence ao grupo SR, e por isso existe o modelo 4 (SR × SA).

## Resumo dos resultados

Métricas no **conjunto de teste**, com os pesos da época de menor loss de validação.

| Modelo | Classes | Exames de teste | Configuração | Acurácia | Acc. balanceada | F1 macro | Baseline* | Melhor época | Tempo |
|---|---|---|---|---|---|---|---|---|---|
| 1 | 4 | 6491 | earlystop15 | 98,01% | 0,9780 | 0,9779 | 36,5% | 10 (de 25) | 59,65 min |
| 1 | 4 | 6491 | descongelamento40 | 96,18% | 0,9553 | 0,9570 | 36,5% | 5 (de 10) | 17,67 min |
| 2 | 2 | 1461 | descongelamento40 | 82,07% | 0,5000 | 0,4508 | 82,1% | 2 (de 7) | 2,89 min |
| 2 | 2 | 1461 | earlystop15 | _sem JSON_ | | | 82,1% | 15 (de 30) | |
| 3 | 4 | 1209 | earlystop15 | 96,20% | 0,8574 | 0,7836 | 89,3% | 14 (de 29) | 13,04 min |
| 3 | 4 | 1209 | descongelamento40 | 91,23% | 0,7186 | 0,6276 | 89,3% | 7 (de 12) | 4,07 min |
| 4 | 2 | 1450 | earlystop15 | 86,83% | 0,7634 | 0,7717 | 81,8% | 24 (de 39) | 21,01 min |
| 4 | 2 | 1450 | descongelamento40 | 84,00% | 0,6372 | 0,6636 | 81,8% | 3 (de 8) | 3,27 min |

\* Baseline = acurácia de um classificador que sempre responde a classe majoritária do conjunto de teste.

Duas configurações de treino:

- `earlystop15`: early stopping com paciência 15. Treino longo (25 a 39 épocas).
- `descongelamento40`: treino curto (7 a 12 épocas, paciência 5 inferida do número de épocas).

Situação de cada modelo (melhor configuração):

- **Modelo 1: bom.** Acurácia de 98,01% e F1 macro de 0,978 no teste (`earlystop15`), consistente com a validação (97,66%).
- **Modelo 2: não aprendeu a tarefa.** Em `descongelamento40` prevê sempre AF (recall de AFIB = 0, acurácia balanceada 0,5). Em `earlystop15` só há curvas, e elas sugerem o mesmo resultado.
- **Modelo 3: parcial.** Acurácia balanceada de 0,857 e F1 macro de 0,784. AT e AVRT agora são reconhecidos (recall 0,76 e 0,75), mas AVRT tem só 4 exames de teste e AT tem precisão de 0,54.
- **Modelo 4: parcial.** Recall de SA de 0,598 (era 0,284), com F1 de SA de 0,62.

Detalhes, matrizes de confusão e análise estão no README de cada modelo.

> **Importante:** as métricas dos modelos 2, 3 e 4 foram calculadas com os exames **da própria classe-grupo real**, e não com os exames que o modelo 1 encaminhou. A avaliação ponta a ponta da cascata (erros do modelo 1 propagados aos especialistas) ainda não foi feita.

### Estimativa grosseira por classe original

Multiplicando o recall do grupo no modelo 1 pelo recall no especialista (supondo que os erros dos dois estágios sejam independentes). **Não substitui a avaliação ponta a ponta.** Usa `earlystop15`, exceto no modelo 2 (só `descongelamento40` tem métricas).

| Classe original | Recall do grupo (modelo 1, `earlystop15`) | Recall no especialista | Estimativa |
|---|---|---|---|
| AFIB | 0,9767 | 0,0000 (descongelamento40) | 0,0000 |
| AF | 0,9767 | 1,0000 (descongelamento40) | 0,9767 |
| SVT | 0,9744 | 0,9524 (earlystop15) | 0,9280 |
| AT | 0,9744 | 0,7561 (earlystop15) | 0,7367 |
| AVRT | 0,9744 | 0,7500 (earlystop15) | 0,7308 |
| ST | 0,9744 | 0,9713 (earlystop15) | 0,9464 |
| SB | 0,9920 | (sem especialista) | 0,9920 |
| SR | 0,9690 | 0,9283 (earlystop15) | 0,8995 |
| SA | 0,9690 | 0,5985 (earlystop15) | 0,5799 |

## Comparação com o experimento anterior

| | Experimento anterior | Este experimento |
|---|---|---|
| Perda | CrossEntropy sem pesos | CrossEntropy com pesos de classe (raiz inversa, teto 20) |
| Scheduler | nenhum (LR fixo 1e-3) | `ReduceLROnPlateau` (fator 0,5, paciência 2) |
| Early stopping | paciência 5 | `descongelamento40`: 5 (inferida) · `earlystop15`: 15 |

| Modelo | Métrica (teste) | Experimento anterior | descongelamento40 | earlystop15 |
|---|---|---|---|---|
| 1 | Acurácia | 97,86% | 96,18% | 98,01% |
| 1 | Acc. balanceada | 0,9752 | 0,9553 | 0,9780 |
| 1 | F1 macro | 0,9763 | 0,9570 | 0,9779 |
| 2 | Acurácia | 82,07% | 82,07% | n/d |
| 2 | Acc. balanceada | 0,5000 | 0,5000 | n/d |
| 2 | F1 macro | 0,4508 | 0,4508 | n/d |
| 3 | Acurácia | 96,28% | 91,23% | 96,20% |
| 3 | Acc. balanceada | 0,6213 | 0,7186 | 0,8574 |
| 3 | F1 macro | 0,6667 | 0,6276 | 0,7836 |
| 4 | Acurácia | 85,66% | 84,00% | 86,83% |
| 4 | Acc. balanceada | 0,6341 | 0,6372 | 0,7634 |
| 4 | F1 macro | 0,6686 | 0,6636 | 0,7717 |

A acurácia balanceada do experimento anterior é a média dos recalls por classe do teste, calculada a partir do README anterior. Modelo 2, `earlystop15`: não há JSON.

O que os números sugerem (uma execução por configuração, semente 42; nada abaixo foi testado isoladamente):

1. **Com paciência 5 (`descongelamento40`), o F1 macro ficou igual ou pior que no experimento anterior** nos modelos 1, 3 e 4. O efeito dos pesos já aparece no modelo 3 (recall de AT de 0,27 para 0,73 e acurácia balanceada de 0,62 para 0,72), mas com 66 ST classificados como AT; no modelo 4 quase não aparece (recall de SA de 0,28 para 0,32). Com paciência 5, o treino termina logo depois da primeira ou da segunda redução de LR (LR final de 5e-4 ou 2,5e-4), possivelmente antes de o scheduler fazer efeito.
2. **O ganho aparece com `earlystop15`.** Os maiores ganhos estão nas classes minoritárias: recall de AT de 0,27 para 0,76, de AVRT de 0,25 para 0,75 e de SA de 0,28 para 0,60. No modelo 1, onde os grupos são equilibrados, a diferença é de 0,15 ponto percentual, ou cerca de 10 exames.
3. **Não dá para separar por completo o efeito dos pesos do efeito do treino mais longo,** porque não houve execução só com pesos e paciência 15, nem só com paciência 15 e sem pesos. Essas duas ablações estão em [próximos passos](#limitações-e-próximos-passos).
4. **O modelo 2 colapsou nos dois experimentos.** Os pesos de classe não resolveram.

## Dados

Cada exame é uma matriz 12 × 5000 (12 derivações, 500 Hz, 10 s) em mV, armazenada em `float16`. Os conjuntos de treino, validação e teste são os mesmos nos 4 modelos e nos dois experimentos. Cada modelo usa apenas os exames das classes da sua tarefa.

Total do modelo 1: 30,288 exames de treino, 6,492 de validação e 6,491 de teste.

Composição por classe original (treino / validação / teste):

| Grupo | Classe original | Treino | Validação | Teste |
|---|---|---|---|---|
| AFIB | AFIB | 1262 | 256 | 262 |
| AFIB | AF | 5557 | 1208 | 1199 |
| GSVT | SVT | 486 | 90 | 84 |
| GSVT | AT | 177 | 32 | 41 |
| GSVT | AVRT | 14 | 6 | 4 |
| GSVT | ST | 4960 | 1078 | 1080 |
| SB | SB | 11052 | 2370 | 2371 |
| SR | SR | 5536 | 1150 | 1186 |
| SR | SA | 1244 | 302 | 264 |

Pesos de classe usados na perda (fórmula: `w = sqrt(N / (K · n_classe))`, com N exames de treino, K classes e teto de 20):

| Modelo | Pesos |
|---|---|
| 1 | AFIB 1,054 · GSVT 1,159 · SB 0,828 · SR 1,057 |
| 2 | AFIB 1,644 · AF 0,783 |
| 3 | SVT 1,703 · AT 2,822 · AVRT 10,033 · ST 0,533 |
| 4 | SR 0,783 · SA 1,651 |

Os rótulos de 4 classes ficam em `y_*.npy` e os de 9 classes em `y_*_9c.npy`, com a ordem dos nomes em `classes_9c.json`.

Pré-processamento (feito no `Dataset`): cada derivação é normalizada individualmente (média 0, desvio 1). **Não há augmentation** (`A_sem_aug`).

## Arquitetura

**ResNet1D**, a mesma ideia da ResNet de imagens, mas com convoluções 1D ao longo do tempo. As 12 derivações entram como 12 canais. Não há pesos pré-treinados: a rede inteira é treinada do zero.

| Etapa | Descrição | Tamanho temporal |
|---|---|---|
| Entrada | Conv1d(12→64, kernel 31, stride 2) + BatchNorm + ReLU + MaxPool(3, stride 2) | 5000 → 2500 → 1250 |
| Estágio 1 | 2 blocos residuais, 64 canais | 1250 |
| Estágio 2 | 2 blocos residuais, 128 canais (stride 2 no primeiro) | 625 |
| Estágio 3 | 2 blocos residuais, 256 canais (stride 2 no primeiro) | 313 |
| Estágio 4 | 2 blocos residuais, 512 canais (stride 2 no primeiro) | 157 |
| Saída | AdaptiveAvgPool1d → Dropout(0,3) → Linear(512 → nº de classes) | - |

Cada bloco residual tem duas Conv1d (kernel 7) com BatchNorm e atalho (convolução 1×1 quando muda o tamanho ou os canais). A única diferença entre os modelos é o número de classes da camada final (4, 2, 4 e 2).

## Configuração de treino

| Parâmetro | Valor |
|---|---|
| Otimizador | Adam, lr inicial = 0,001 |
| Perda | CrossEntropyLoss com pesos de classe (raiz inversa da frequência, teto 20) |
| Scheduler | `ReduceLROnPlateau`: fator 0,5, paciência 2, lr mínimo 1e-6 (monitora, ao que tudo indica, a loss de validação) |
| Batch size | 32 (presumido igual ao experimento anterior) |
| Épocas máximas | 100 (presumido igual ao experimento anterior) |
| Early stopping | `earlystop15`: paciência 15 · `descongelamento40`: paciência 5 (inferida: épocas executadas = melhor época + 5 em todos os modelos) |
| Seleção do modelo | pesos da época com menor loss de validação |
| Augmentation | nenhuma (`A_sem_aug`) |
| Semente | `torch.manual_seed(42)` (presumido) |
| Ambiente | Google Colab, GPU (T4) (presumido) |

> **Atenção:** o nome `descongelamento40` vem dos arquivos de resultado, e os JSONs não registram o que ele significa. Como a rede é treinada do zero, sem camadas congeladas, o nome não parece descrever congelamento. **Confirme e descreva aqui** o que distingue essa configuração. Os itens marcados como "presumido" ou "inferida" também precisam de confirmação.

O mesmo código de treino é usado nos 4 modelos; a escolha do modelo é feita pela variável `TAREFA` do notebook.

## Documentação por modelo

- [Modelo 1: AFIB, GSVT, SB, SR](modelo1/README.md)
- [Modelo 2: AFIB × AF](modelo2/README.md)
- [Modelo 3: SVT, AT, AVRT, ST](modelo3/README.md)
- [Modelo 4: SR × SA](modelo4/README.md)

## Como reproduzir

O treino é feito no notebook `ECG_Pratica_3_CNN1D_cascata.ipynb` (Google Colab, GPU). **O notebook do experimento anterior não tem pesos de classe nem scheduler**; use a versão com essas alterações (não incluída aqui).

1. Coloque no Google Drive, em `EcgArritmia/`, a pasta `sinais_ecg_4c_compacto` (com os blocos `X_*.npy.xz`, os `y_*.npy`, os `y_*_9c.npy`, os `registros_*.csv` e o `classes_9c.json`).
2. Na célula de configurações, defina a tarefa:
   ```python
   TAREFA = "modelo1"   # "modelo1" | "modelo2" | "modelo3" | "modelo4"
   ```
3. Execute a célula 0 (copia os dados e remonta os `X_*.npy`) e depois as demais em sequência.
4. Os resultados são salvos em `EcgArritmia/resultados/` com o prefixo `sinais_ecg_4c_<tarefa>_resnet1d_A_sem_aug_pesos-raiz_inversa_lrsched_<config>`, com `<config>` = `earlystop15` ou `descongelamento40`:

| Arquivo | Conteúdo |
|---|---|
| `*.json` | configuração, pesos de classe, scheduler, contagem por classe, métricas de validação e teste, histórico por época (inclui `lr`) |
| `*.pt` | pesos do melhor modelo (`state_dict`) |
| `*_loss.png`, `*_acuracia.png`, `*_lr.png` | curvas de treino, validação e learning rate |
| `*_matriz_validacao.png`, `*_matriz_teste.png` | matrizes de confusão (% da classe real e contagem) |

Para o modelo 1 o código lê o `y_*.npy` de 4 classes. Para os demais, filtra os exames pelo `y_*_9c.npy` e reindexa os rótulos de 0 a N−1 (e confere se os rótulos de 9 classes são consistentes com os de 4).

## Estrutura do repositório

```
.
├── README.md
├── modelo1/   README.md + earlystop15/ e descongelamento40/ (loss, acurácia, lr, matrizes)
├── modelo2/   (earlystop15/ só tem as curvas)
├── modelo3/
└── modelo4/
```

## Limitações e próximos passos

- **Cascata não avaliada ponta a ponta:** falta um script de inferência que rode o modelo 1 e encaminhe ao especialista, medindo F1 e matriz de confusão nas 9 classes originais.
- **Modelo 2 ainda não aprendeu a tarefa.** Falta o JSON e as matrizes de `earlystop15` para confirmar. Acompanhar AUC e acurácia balanceada por época, e ajustar o limiar de decisão.
- **Uma única execução por configuração** (semente 42): não há média nem desvio entre execuções, então diferenças pequenas não devem ser interpretadas (por exemplo, a diferença de 0,15 ponto percentual no modelo 1).
- **Classes muito pequenas no modelo 3:** AVRT tem 14 exames de treino e 4 de teste; AT tem 41 de teste. Reportar intervalos de confiança.
- **Seleção de época pela loss de validação.** Com perda ponderada e classes pequenas, ela é ruidosa (modelo 3, `descongelamento40`) e pode escolher épocas de pior acurácia. Registrar F1 macro e acurácia balanceada por época e comparar critérios.
- **Quedas de acurácia de validação nas primeiras épocas** (modelos 1, 3 e 4, LR = 1e-3). Testar LR inicial menor, warmup ou gradient clipping.
- **Ablações que faltam** para separar os efeitos: só pesos com paciência 15, e só paciência 15 sem pesos.
- **Augmentation** (só nas classes minoritárias) e **features de HRV** para o modelo 4 não foram testados.
- Não há avaliação em dados externos à base.

## Glossário

| Sigla | Significado |
|---|---|
| AFIB | Fibrilação atrial |
| AF | Flutter atrial |
| SVT | Taquicardia supraventricular |
| AT | Taquicardia atrial |
| AVRT | Taquicardia por reentrada atrioventricular |
| ST | Taquicardia sinusal |
| SB | Bradicardia sinusal |
| SR | Ritmo sinusal |
| SA | _(preencher conforme a nomenclatura da base; provavelmente arritmia sinusal)_ |
| GSVT | Grupo criado neste projeto: SVT, AT, AVRT e ST |
