# Notas resumindo o que aprendi sobre o EDA
- o dataset está divindo proporcionalmente quarks e gluons (classe 0 e 1)

- pt é transverse momentum e m0 é invariant mass, é importante essas variaveis nao serem constantes


#### pixels nao zero:
- gluons: média ~728, mediana 709
- quarks: média ~635, mediana 616
por conta da diferença de ~15%, gluons tem jets mais espalhados. Como cada pixel ativo vira um nó, gluons terão grafos maiores em média

- sparsity de ~95%, o que justifica o point cloud, trabalhar com todos os pixels seria insuficiente já que é quase tudo 0

#### mapas de intensidade média
por ser uma parte mais de física, copio o que claude disse:
- Gluons têm halo de deposição visivelmente mais espalhado em ECAL e HCAL
- Quarks têm energia mais concentrada no núcleo central
- Canal Tracks é mais esparso e a diferença é menos visível
- O mapa de diferença mostra claramente que gluons depositam mais energia na periferia do jet

#### intensidade integrada por canal
- gluons dispõem mais energia total em todos os canais
- canal tracks tem separação mais visível entre eles

# Notas do que foi implementado após o EDA
- função `to_hwc`, que converte imagens de ChannelHeighWidth para HeightWidhtChannel quando necessário, verificando o shape antes de transpor
- `graph_const` que converte uma imagem em objeto `Data` do pytorch geometric
- `build_graphs` itera sob o dataset inteiro chamando `graph_const` para cada evento, com sistema de cache

# EDA de grafos

#### nós por grafo
- gluons têm mais nós que quarks, distribuição deslocada para a direita
- **confounder importante:** número de nós sozinho já separa as classes — o modelo pode aprender a classificar por multiplicidade em vez de topologia

#### arestas por grafo
- segue o mesmo padrão dos nós — proporcional ao número de nós com kNN fixo k=8
- não adiciona informação discriminativa além do que os nós já mostram

#### grau médio
- idêntico entre classes (~8.5)
- esperado: com kNN fixo cada nó tem aproximadamente o mesmo número de vizinhos independente da classe

#### componentes conectados
- maioria dos grafos tem 1-2 componentes
- gluons têm ligeiramente mais componentes — pixels periféricos isolados no espaço (η, φ) formam componentes separados
- relevante para o modelo: EdgeConv não passa mensagens entre componentes desconectados, então pixels isolados contribuem apenas via pooling

#### conclusão
o sinal discriminativo está na estrutura espacial e na multiplicidade de constituintes.
a representação em grafo captura isso naturalmente, mas o número de nós sozinho já separa as classes, 
o modelo precisará aprender além dessa feature trivial para ser genuinamente útil.

# Baseline Logistic Regression

- mean pooling das features de nó de cada grafo, gerando um vetor de 6 features por grafo
- treinou LogisticRegression do sklearn nesses vetores

#### resultados no teste:
- Accuracy: 0.512
- F1 Score: 0.509
- AUC: 0.515

AUC de 0.515 é praticamente aleatório, features agregadas por mean pool não carregam informação discriminativa suficiente.
a normalização por evento faz com que a média das intensidades seja quase constante entre eventos, então o classificador linear não tem sinal para aprender.
isso justifica o uso de GNN,  a estrutura relacional entre os nós, que o mean pool destrói, é onde o sinal discriminativo está.

# Baseline CNN (SmallCNN)

- 3 camadas conv (16 → 32 → 64), AdaptiveAvgPool, MLP classificador
- treinado nas imagens originais 125x125x3 com os mesmos índices de split
- 8 épocas, early stopping patience=3, Adam lr=1e-4

#### resultados no teste:
- Accuracy: 0.6866
- F1: 0.6965
- AUC: 0.7471

AUC de 0.747 é significativamente melhor que o baseline linear (0.515),
confirmando que há sinal discriminativo nas imagens.
esse é o número que a GNN precisa superar para justificar a representação em grafo.