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