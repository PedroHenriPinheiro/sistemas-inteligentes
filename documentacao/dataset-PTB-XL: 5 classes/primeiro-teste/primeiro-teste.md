# Classificação de ECG (PTB-XL) — documentação dos modelos e do filtro

Dos 15 modelos avaliados no Orange, 4 redes neurais passam no corte de **AUC ≥ 0,70** no teste de 500 ECGs. Os outros 11 saem por ficarem abaixo desse corte ou por colapsarem em uma única classe.

## Dados

Subconjunto balanceado de imagens de ECG derivado do dataset baseado no PTB-XL.

| Item | Valor |
| --- | --- |
| Classes | CD, HYP, MI, NORM, STTC |
| Treino | 400 imagens por classe (2.000 no total) |
| Teste | 100 imagens por classe (500 no total) |
| Total | 2.500 imagens |
| Linha de base (acaso) | CA = 0,200 (5 classes balanceadas) |

### Distribuição dos dados (t-SNE)

![t-SNE dos 2.000 exemplos de treino](docs/tsne.png)

Parâmetros: normalização + PCA com 20 componentes, inicialização PCA, distância euclidiana, perplexidade 30.

O t-SNE não mostra agrupamentos por classe: CD, HYP, MI, NORM e STTC aparecem misturados em toda a nuvem. As classes se sobrepõem bastante no espaço de entrada, o que antecipa o teto baixo de desempenho dos modelos.

## Resultados no teste (500 imagens)

Métricas médias sobre as 5 classes, geradas pelo widget Predictions do Orange, ordenadas por AUC.

![Predictions — Orange](docs/predictions.png)

| # | Modelo | AUC | CA | F1 | Prec | Recall | MCC |
| --- | --- | --- | --- | --- | --- | --- | --- |
| 1 | NN · Identity · Adam · 20,20,20,20,20 | 0,744 | 0,424 | 0,422 | 0,423 | 0,424 | 0,281 |
| 2 | NN · Identity · Adam · 10 × 8 camadas | 0,741 | 0,424 | 0,424 | 0,424 | 0,424 | 0,280 |
| 3 | NN · Identity · Adam · 100,100,100 | 0,735 | 0,438 | 0,433 | 0,434 | 0,438 | 0,299 |
| 4 | NN · ReLU · Adam · 100,100,100 | 0,708 | 0,392 | 0,390 | 0,391 | 0,392 | 0,240 |
| 5 | KNN · Manhattan · por distância · k=100 | 0,676 | 0,326 | 0,317 | 0,333 | 0,326 | 0,161 |
| 6 | KNN · Euclidiana · uniforme · k=100 | 0,673 | 0,328 | 0,320 | 0,328 | 0,328 | 0,162 |
| 7 | KNN · Manhattan · uniforme · k=100 | 0,673 | 0,328 | 0,319 | 0,330 | 0,328 | 0,163 |
| 8 | NN · ReLU · Adam · 20 × 6 camadas | 0,651 | 0,362 | 0,363 | 0,364 | 0,362 | 0,203 |
| 9 | NN · tanh · L-BFGS-B · 400,10,400,10 (execução A) | 0,649 | 0,334 | 0,330 | 0,332 | 0,334 | 0,168 |
| 10 | NN · tanh · L-BFGS-B · 400,10,400,10 (execução B) | 0,627 | 0,312 | 0,309 | 0,310 | 0,312 | 0,140 |
| 11 | KNN · Manhattan · por distância · k=5 | 0,625 | 0,322 | 0,322 | 0,325 | 0,322 | 0,153 |
| 12 | KNN · Euclidiana · uniforme · k=5 | 0,620 | 0,304 | 0,298 | 0,300 | 0,304 | 0,131 |
| 13 | KNN · Manhattan · uniforme · k=5 | 0,619 | 0,310 | 0,305 | 0,308 | 0,310 | 0,138 |
| 14 | NN · Logistic · SGD · 20,100,100,20 | 0,488 | 0,200 | 0,067 | 0,040 | 0,200 | 0,000 |
| 15 | NN · Logistic · SGD · 100,20,20,100 | 0,482 | 0,200 | 0,067 | 0,040 | 0,200 | 0,000 |

## Filtro aplicado e justificativa

**Regra proposta:** manter apenas modelos com AUC ≥ 0,70 e que não colapsem em uma única classe. O corte cai num intervalo natural da tabela: há um salto entre o 4º modelo (AUC 0,708) e o 5º (0,676), sem nenhum modelo no meio.

| Grupo removido | Modelos (#) | Evidência | Justificativa |
| --- | --- | --- | --- |
| Colapso em classe única | NN Logistic · SGD (14, 15) | CA = 0,200, MCC = 0,000, F1 = 0,067, Prec = 0,040 | Os valores coincidem com o de um modelo que prevê sempre a mesma classe: acerto igual ao acaso (20%) e MCC nulo. Não aprenderam nada útil. |
| KNN, todas as variações | KNN (5, 6, 7, 11, 12, 13) | AUC 0,619–0,676; CA 0,304–0,328 | Métrica (Manhattan ou Euclidiana) e ponderação (uniforme ou por distância) mudam a CA em menos de 0,03 para o mesmo k, então o ajuste fino não resolve. Aumentar k de 5 para 100 ajuda (AUC +0,05), mas continua abaixo do corte. Coerente com o t-SNE: vizinhos próximos de classes diferentes. |
| NN tanh com L-BFGS-B | NN tanh (9, 10) | AUC 0,649 e 0,627 | Abaixo do corte, e a mesma configuração rendeu resultados diferentes nas duas execuções, o que indica instabilidade. |
| NN ReLU 20 × 6 camadas | NN ReLU (8) | AUC 0,651; CA 0,362 | Abaixo do corte. A versão ReLU mais larga e rasa (100,100,100) teve AUC 0,708, então a rede profunda e estreita não compensa. |

### Modelos mantidos

- NN · Identity · Adam · 20,20,20,20,20 (#1)
- NN · Identity · Adam · 10 × 8 camadas (#2)
- NN · Identity · Adam · 100,100,100 (#3)
- NN · ReLU · Adam · 100,100,100 (#4)

As diferenças entre os três Identity são pequenas (AUC 0,735–0,744; CA 0,424–0,438). Com 500 amostras, o erro padrão da acurácia é de cerca de 0,022, então essas diferenças não são estatisticamente distinguíveis. Além disso, uma rede com ativação identity equivale a um modelo linear, qualquer que seja a profundidade, o que explica resultados tão parecidos.

**Sugestão:** manter o 100,100,100 como representante dos Identity (maior CA, F1 e MCC) e o 20,20,20,20,20 (maior AUC), e usar o ReLU como referência não linear.

## Limitações e pontos em aberto

- O melhor modelo acerta 43,8% das imagens, cerca de 2,2 vezes o acaso (20%). O filtro escolhe os menos fracos; o desempenho geral ainda é baixo, e o t-SNE sugere que o limite pode estar na representação dos dados e não só no classificador.
- Registrar como as imagens foram transformadas em atributos antes dos modelos (extração de características, tamanho da imagem, embeddings).
- Os nomes das execuções 9 e 10 são idênticos; anotar o que as diferencia (semente, iterações, regularização).
- A configuração do modelo 2 aparece truncada na tela (10 neurônios por camada, 8 camadas visíveis); confirmar o número exato de camadas.
- Todos os números vêm de um único conjunto de teste de 500 imagens, sem validação cruzada.