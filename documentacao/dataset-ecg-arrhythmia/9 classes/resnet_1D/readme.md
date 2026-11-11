# Classificação de arritmias em ECG de 12 derivações com CNN 1D (cascata de modelos)

Classificação de arritmias direto no sinal de ECG (12 derivações × 5000 amostras, 500 Hz) usando uma **ResNet1D** treinada do zero, organizada como uma **cascata de modelos**: um modelo geral separa grupos de ritmos e modelos especialistas refinam os grupos que reúnem várias classes originais.

Base de dados: PhysioNet, *A large scale 12-lead ECG database for arrhythmia study*.

## Sumário

- [Visão geral da cascata](#visão-geral-da-cascata)
- [Resumo dos resultados](#resumo-dos-resultados)
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

Métricas no **conjunto de teste**, com os pesos da melhor época (menor loss de validação).

| Modelo | Classes | Exames de teste | Acurácia | F1 macro | Baseline* | Melhor época | Tempo de treino |
|---|---|---|---|---|---|---|---|
| 1 | 4 | 6491 | **97,86%** | **0,9763** | 36,5% | 18 (de 23) | 58,5 min |
| 2 | 2 | 1461 | 82,07% | 0,4508 | 82,1% | 17 (de 22) | 10,8 min |
| 3 | 4 | 1209 | 96,28% | 0,6667 | 89,3% | 8 (de 13) | 5,7 min |
| 4 | 2 | 1450 | 85,66% | 0,6686 | 81,8% | 8 (de 13) | 7,0 min |

\* Baseline = acurácia de um classificador que sempre responde a classe majoritária do conjunto de teste.

Situação de cada modelo:

- **Modelo 1: bom.** Acurácia de 97,9% no teste, consistente com a validação (98,0%).
- **Modelo 2: não aprendeu a tarefa.** Prevê sempre AF; a acurácia é igual ao baseline e o recall de AFIB é 0%.
- **Modelo 3: parcial.** Bom em SVT e ST, fraco em AT e AVRT (classes pequenas).
- **Modelo 4: parcial.** Detecta bem SR (recall 98,4%), mas só 28,4% dos SA.

Detalhes, matrizes de confusão e análise estão no README de cada modelo.

> **Importante:** as métricas dos modelos 2, 3 e 4 foram calculadas com os exames **da própria classe-grupo real**, e não com os exames que o modelo 1 encaminhou. A avaliação ponta a ponta da cascata (erros do modelo 1 propagados aos especialistas) ainda não foi feita.

## Dados

Cada exame é uma matriz 12 × 5000 (12 derivações, 500 Hz, 10 s) em mV, armazenada em `float16`. Os conjuntos de treino, validação e teste são os mesmos nos 4 modelos. Cada modelo usa apenas os exames das classes da sua tarefa.

Total do modelo 1: 30.288 exames de treino, 6.492 de validação e 6.491 de teste.

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

Os rótulos de 4 classes ficam em `y_*.npy` e os de 9 classes em `y_*_9c.npy`, com a ordem dos nomes em `classes_9c.json`.

Pré-processamento (feito no `Dataset`): cada derivação é normalizada individualmente (média 0, desvio 1). **Não há augmentation.**

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
| Otimizador | Adam, lr = 0,001 |
| Perda | CrossEntropyLoss, sem pesos por classe |
| Batch size | 32 |
| Épocas máximas | 100 |
| Early stopping | paciência 5, monitorando a loss de validação |
| Seleção do modelo | pesos da época com menor loss de validação |
| Augmentation | nenhuma (`A_sem_aug`) |
| Semente | `torch.manual_seed(42)` |
| Ambiente | Google Colab, GPU (T4) |

O mesmo código de treino é usado nos 4 modelos; a escolha do modelo é feita pela variável `TAREFA` do notebook.

## Documentação por modelo

- [Modelo 1: AFIB, GSVT, SB, SR](modelo1/README.md)
- [Modelo 2: AFIB × AF](modelo2/README.md)
- [Modelo 3: SVT, AT, AVRT, ST](modelo3/README.md)
- [Modelo 4: SR × SA](modelo4/README.md)

## Como reproduzir

O treino é feito no notebook `ECG_Pratica_3_CNN1D_cascata.ipynb` (Google Colab, GPU).

1. Coloque no Google Drive, em `EcgArritmia/`, a pasta `sinais_ecg_4c_compacto` (com os blocos `X_*.npy.xz`, os `y_*.npy`, os `y_*_9c.npy`, os `registros_*.csv` e o `classes_9c.json`).
2. Na célula de configurações, defina a tarefa:
   ```python
   TAREFA = "modelo1"   # "modelo1" | "modelo2" | "modelo3" | "modelo4"
   ```
3. Execute a célula 0 (copia os dados e remonta os `X_*.npy`) e depois as demais em sequência.
4. Os resultados são salvos em `EcgArritmia/resultados/` com o prefixo `sinais_ecg_4c_<tarefa>_resnet1d_A_sem_aug`:

| Arquivo | Conteúdo |
|---|---|
| `*.json` | configuração, contagem por classe, métricas de validação e teste, histórico por época |
| `*.pt` | pesos do melhor modelo (`state_dict`) |
| `*_loss.png`, `*_acuracia.png` | curvas de treino e validação |
| `*_matriz_validacao.png`, `*_matriz_teste.png` | matrizes de confusão (% da classe real e contagem) |

Para o modelo 1 o código lê o `y_*.npy` de 4 classes. Para os demais, filtra os exames pelo `y_*_9c.npy` e reindexa os rótulos de 0 a N−1 (e confere se os rótulos de 9 classes são consistentes com os de 4).

## Estrutura do repositório

```
.
├── README.md
├── ECG_Pratica_3_CNN1D_cascata.ipynb
├── modelo1/   README.md + figuras (loss, acurácia, matrizes)
├── modelo2/
├── modelo3/
└── modelo4/
```

## Limitações e próximos passos

- **Cascata não avaliada ponta a ponta:** falta um script de inferência que rode o modelo 1 e encaminhe ao especialista, medindo F1 e matriz de confusão nas 9 classes originais.
- **Modelos 2, 3 e 4 precisam melhorar** (ver o README de cada um). Pontos em comum: desbalanceamento de classes sem nenhum tratamento (sem pesos na perda e sem augmentation) e classes muito pequenas no modelo 3 (AVRT tem 14 exames de treino e 4 de teste).
- **Uma única execução por modelo** (semente 42): não há média nem desvio entre execuções, então diferenças pequenas não devem ser interpretadas.
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
| SA | irregularidade sinusal |
| GSVT | Grupo criado neste projeto: SVT, AT, AVRT e ST |