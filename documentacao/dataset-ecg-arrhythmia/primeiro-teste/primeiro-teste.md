from pathlib import Path

readme = r"""# Atividade Prática 2 — Organização do Dataset e Planejamento dos Experimentos

**Disciplina:** Aplicações em Sistemas Inteligentes  
**Entrega:** 1ª entrega — 20/10/2026  
**Integrantes:** Nycaio Bezerra (2220230) e Pedro Henrique (2420509)

## 1. Dataset

**Base utilizada:** *A large scale 12-lead electrocardiogram database for arrhythmia study* — PhysioNet, v1.0.0.

- 45.152 ECGs de 12 derivações, com duração de 10 segundos e frequência de amostragem de 500 Hz.
- Formato original WFDB (`.hea` + `.mat`), totalizando aproximadamente 5,3 GB.
- Cada registro pertence a um paciente diferente; portanto, a divisão por registro também separa os pacientes entre treino, validação e teste.
- Os diagnósticos são representados por códigos SNOMED-CT na linha `#Dx` dos cabeçalhos. A base contém 94 códigos diferentes, e aproximadamente metade dos exames apresenta mais de um diagnóstico (até 12).

### 1.1. Definição das classes

Como um ECG pode conter vários diagnósticos, para esta tarefa foi utilizado apenas o diagnóstico referente ao **ritmo cardíaco**, atribuindo uma única classe a cada imagem.

| Classe | Ritmo cardíaco | Imagens | Proporção | SB ÷ classe |
|---|---|---:|---:|---:|
| SB | Bradicardia sinusal | 15.793 | 36,5% | 1× |
| AF | Flutter atrial | 7.964 | 18,4% | 2,0× |
| SR | Ritmo sinusal (normal) | 7.872 | 18,2% | 2,0× |
| ST | Taquicardia sinusal | 7.118 | 16,4% | 2,2× |
| SA | Irregularidade sinusal | 1.810 | 4,2% | 8,7× |
| AFIB | Fibrilação atrial | 1.780 | 4,1% | 8,9× |
| SVT | Taquicardia supraventricular | 660 | 1,5% | 24× |
| AT | Taquicardia atrial | 250 | 0,6% | 63× |
| AVRT | Taquicardia por reentrada atrioventricular | 24 | 0,06% | 658× |
| **Total** | | **43.271** | **100%** | |

Foram descartados 1.881 exames:
- 949 sem nenhum ritmo identificado;
- 838 com ritmos conflitantes (por exemplo, fibrilação e ritmo sinusal);
- 94 com sinal vazio/NaN ou cabeçalho ilegível.

Nos 321 exames com dois ritmos semelhantes, prevaleceu o diagnóstico mais específico (por exemplo, `SR + SA → SA` e `ST + AT → AT`).

### 1.2. Conjunto utilizado

Foi utilizada a base completa, sem seleção de subconjunto: **43.271 ECGs com ritmo válido**. A conversão para PNG gerou aproximadamente 1,8 GB de imagens, volume considerado viável para treinamento no Google Colab.

A base é fortemente desbalanceada:
- As quatro maiores classes (SB, AF, SR e ST) representam 89,5% das imagens.
- SVT, AT e AVRT representam apenas 2,2%.
- A classe SB possui 658 vezes mais exames que AVRT.

### 1.3. Divisão dos dados

Foi realizada uma divisão estratificada de **70% para treino, 15% para validação e 15% para teste**, usando `seed = 42`, igual para todos os integrantes.

| Classe | Treino | Validação | Teste | Total |
|---|---:|---:|---:|---:|
| AF | 5.575 | 1.195 | 1.194 | 7.964 |
| AFIB | 1.246 | 267 | 267 | 1.780 |
| AT | 176 | 37 | 37 | 250 |
| AVRT | 17 | 3 | 4 | 24 |
| SA | 1.268 | 270 | 272 | 1.810 |
| SB | 11.058 | 2.372 | 2.363 | 15.793 |
| SR | 5.510 | 1.183 | 1.179 | 7.872 |
| ST | 4.987 | 1.064 | 1.067 | 7.118 |
| SVT | 462 | 99 | 99 | 660 |
| **Total** | **30.299** | **6.490** | **6.482** | **43.271** |

A lista exata dos registros de cada conjunto está em `dataset_ecg/divisao.csv`.

### 1.4. Conversão de WFDB para imagem

O processo de conversão foi definido da seguinte forma:

1. Leitura do registro com `wfdb.rdrecord()`, obtendo o sinal em mV (5.000 amostras × 12 derivações).
2. Aplicação de filtro passa-banda Butterworth de 0,5 a 40 Hz, para reduzir a oscilação da linha de base e o ruído de alta frequência.
3. Empilhamento vertical das 12 derivações (`I`, `II`, `III`, `aVR`, `aVL`, `aVF`, `V1`–`V6`), mostrando os 10 segundos completos de cada uma e mantendo a mesma escala em mV para preservar a amplitude.
4. Geração de imagens PNG de 512 × 512 pixels, com traçado preto sobre fundo branco.
5. Redimensionamento para 224 × 224 pixels durante o treinamento.

## 2. Estratégias de Data Augmentation

As transformações foram definidas no arquivo `transformacoes.py`, compartilhado por toda a equipe. O augmentation é aplicado **somente ao conjunto de treino**. Validação e teste utilizam a configuração A.

### 2.1. Transformações proibidas para ECG

- **Flip horizontal:** inverteria o tempo, fazendo a onda P aparecer depois do complexo QRS.
- **Flip vertical:** inverteria a polaridade das ondas, alterando características relevantes para o diagnóstico.
- **Rotações e recortes grandes:** poderiam cortar batimentos e distorcer os intervalos RR, importantes para classificar o ritmo.

### 2.2. Configurações planejadas

| Configuração | Transformações |
|---|---|
| **A — Sem augmentation** | Resize 224 × 224, `ToTensor` e `Normalize` com média e desvio do ImageNet. |
| **B — Leve** | Translação de ±2% e brilho de ±10%. |
| **C — Moderado** | Translação de ±4%, escala de 0,95–1,05, brilho de ±20% e contraste de ±20%. |
| **D — Pesado** | Rotação de ±3°, translação de ±6%, escala de 0,90–1,10, brilho e contraste de ±30%, Gaussian Blur (kernel 3, sigma 0,1–1,0, `p = 0,3`) e Random Erasing (1–4% da área, `p = 0,3`). |

Nas transformações geométricas, as novas áreas são preenchidas com branco (`fill = 255`). O Random Erasing também utiliza branco, simulando pequenos trechos com perda de sinal.

### 2.3. Hiperparâmetros fixos

| Parâmetro | Valor |
|---|---|
| Batch size | 32 |
| Otimizador | Adam |
| Learning rate | 0,001 |
| Épocas máximas | 100 |
| Early Stopping | Paciência 5, monitorando a loss de validação e restaurando o melhor modelo |
| Transfer Learning | Pesos do ImageNet, backbone congelado e treinamento apenas da camada de saída |
| Seed | 42 |

### 2.4. Tratamento do desbalanceamento

A base completa será mantida, sem descartar exames por undersampling nem duplicar imagens. O primeiro treinamento foi realizado sem correção do desbalanceamento, servindo como referência (*baseline*).

A estratégia planejada para os experimentos é utilizar **pesos por classe na função de perda**, por meio de `CrossEntropyLoss(weight=...)`, com pesos inversamente proporcionais à frequência de cada classe no conjunto de treino. Assim, erros em classes raras terão maior custo.

Além da acurácia, serão analisados o **recall por classe** e o **F1 macro**, pois a acurácia pode ser dominada pela classe majoritária SB.

## 3. Distribuição das arquiteturas

Cada integrante treinará três arquiteturas nas quatro configurações de augmentation, totalizando 12 treinamentos por pessoa e 24 no total. O trabalho contempla cinco famílias de redes: EfficientNet, ResNet, DenseNet, VGG e MobileNet.

| Integrante | Matrícula | Arquitetura 1 | Arquitetura 2 | Arquitetura 3 |
|---|---|---|---|---|
| Nycaio Bezerra | 2220230 | EfficientNet-B2 | ResNet50 | DenseNet121 |
| Pedro Henrique | 2420509 | VGG16 | MobileNetV3-Large | ResNet18 |

## 4. Resultados parciais

### 4.1. EfficientNet-B2 — Configuração A

O primeiro treinamento foi concluído no Google Colab, sem correção do desbalanceamento.

| Métrica | Treino | Validação | Teste |
|---|---:|---:|---:|
| Acurácia | 72,45% | 77,60% | 77,12% |
| Loss | 0,803 | 0,665 | — |
| F1 macro | — | 0,446 | 0,446 |
| F1 ponderado | — | 0,749 | 0,744 |

- **Melhor época:** 9
- **Épocas executadas:** 14
- **Early Stopping:** acionado após 5 épocas sem melhora
- **Tempo de treinamento:** 44,3 minutos

### 4.2. Desempenho por classe no teste

| Classe | Imagens de teste | Precisão | Recall | F1 |
|---|---:|---:|---:|---:|
| SB | 2.363 | 0,886 | 0,970 | 0,926 |
| ST | 1.067 | 0,748 | 0,880 | 0,808 |
| SR | 1.179 | 0,687 | 0,723 | 0,704 |
| SVT | 99 | 0,552 | 0,697 | 0,616 |
| AF | 1.194 | 0,673 | 0,668 | 0,671 |
| SA | 272 | 0,644 | 0,173 | 0,273 |
| AFIB | 267 | 0,133 | 0,008 | 0,014 |
| AT | 37 | 0,000 | 0,000 | 0,000 |
| AVRT | 4 | 0,000 | 0,000 | 0,000 |

### 4.3. Observações iniciais

- A acurácia de aproximadamente 77% não representa, sozinha, o desempenho equilibrado do modelo. O F1 macro é 0,446: o modelo classifica melhor as classes majoritárias (SB, ST, SR e AF), mas quase nunca prevê algumas classes raras (AFIB, AT e AVRT).
- **AFIB é confundida com AF:** 189 das 267 amostras de fibrilação atrial foram classificadas como flutter atrial. As duas classes correspondem a arritmias atriais, e AF possui cerca de 4,5 vezes mais exemplos.
- **SA é confundida com SR:** 150 das 272 amostras de irregularidade sinusal foram classificadas como ritmo sinusal normal, uma diferença sutil nas imagens.
- **Não foi observado overfitting:** a loss de validação permaneceu abaixo da loss de treino em todas as épocas. Isso é esperado porque o dropout da EfficientNet atua somente durante o treino.
- As curvas estabilizaram já na segunda época, sugerindo **underfitting**: com o backbone congelado, somente a camada final é treinada.

### 4.4. Próximos passos

1. Aplicar os pesos por classe na função de perda.
2. Executar as configurações de augmentation B, C e D.
3. Comparar os resultados com o baseline da configuração A.

---

## Estrutura de arquivos mencionada

```text
dataset_ecg/
└── divisao.csv

transformacoes.py
```

> **Observação:** este README documenta a primeira entrega e os resultados parciais apresentados nela. Os experimentos seguintes ainda serão executados e comparados com o baseline.
"""
path = Path("/mnt/data/README.md")
path.write_text(readme, encoding="utf-8")
print(f"README criado: {path}")
