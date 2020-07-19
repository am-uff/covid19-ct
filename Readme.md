## Como reproduzir o artigo?

O diretório `notebooks` contém todos os passos para repdodução dos experimentos.
Para simplificar o trabalho em relação a instalação das dependências e bibliotecas,
sugerimos usar o [Google Colab](https://colab.research.google.com/github/googlecolab/colabtools/blob/master/notebooks/colab-github-demo.ipynb). Dessa forma, todas as dependências serão automaticamente satisfeitas.


## Sobre as CNNs propostas para a detecção da Covid-19.

### Método 1:

O Método 1 proposto neste trabalho se baseia na técnica de transferência de aprendizado
utilizando uma base convolucional de uma arquitetura `VGG-16` treinada com imagens
da `Imagenet` [Deng et al. 2009] combinada com módulos `Inception` e camadas completamente
conectadas para classificar imagens de tomografia computadorizada indicando se
o paciente tem COVID-19 ou não.

A base convolucional da rede corresponde as camadas convolucionais conv1 a
conv5 ilustradas na figura 1, seguidas de três módulos inception, com a mesma estrutura
do módulo descrito na figura 2. Na sequência dos módulos inception foi adicionada uma
camada de Global Average Pooling [Lin et al. 2013] para achatar os dados de entrada
para uma sequência de três camadas densas, as duas primeiras com 256 neurônios e a
terceira com apenas uma unidade com a funçâo de ativação sigmoide para classificar se
a imagem é de uma paciente com COVID-19. Como otimizador foi escolhido o Adam
[Kingma and Ba 2014] com uma taxa de aprendizagem de 2e-10 e a função custo escolhida
para uso no treinamento da rede foi a entropia cruzada.

Antes do treinamento da rede, os pesos da base convolucional da
VGG-16 pre-treinada com a `Imagenet`  são congelados para que a base convolucional possa
ser usada na extração de features, enquanto que os módulos inception e as camadas densas tem seus pesos atualizados durante o treinamento com base nas imagens de tomografia de tórax disponíveis nos dados de treinamento.

__Notebook para o método 1__: [transfer_learning_vgg_inception_modules.ipynb](notebooks/transfer_learning_vgg_inception_modules.ipynb)

## Sobre as redes propostas para a detecção da Covid-19.

### Método 2:

O segundo método proposto neste trabalho, utiliza os pesos de uma rede pré-treinada com imagens de Raio-X de tórax [Wang et al. 2017]. Esse pré-treinamento foi realizado utilizando uma rede `DenseNet` de 121 camadas. Foi utilizada uma configuração semelhante à rede original publicada em [Rajpurkar et al. 2017]. As camadas do topo da rede foram congeladas e os pesos do pré-treinamento carregados.
A camada de classificação foi substituída, onde foram acrescentadas uma camada
de Global Average Pooling e duas camadas densas contendo 64 e 32 neurônios respectivamente.

As camadas densas foram intercaladas por uma camada de dropout, configurada
com uma probabilidade de 0.5. Para as camadas densas foi utilizada a função de ativação
não linear ReLU e na última camada de classificação optou-se por utilizar a função sigmoide.
Para o treinamento foram utilizadas 30 epochs e na otimização aplicou-se Adam
com um _learning rate_ de 0.001. 

A diferença do método proposto, está no fato de explorar informações de imagens médicas de mesmo domínio ao invés de usar, por exemplo, os pesos aprendidos com o treinamento em imagens de outros domínios (`Imagenet`), menos relevantes ao problema.

__Notebook para o método 2__: [transfer_learning_densenet121.ipynb](notebooks/transfer_learning_densenet121.ipynb)


## Resultados Obtidos

Para o método 1 obtivemos os seguintes resultados: __acurácia (0.74)__, __F1-Score (0.75)__ e __AUC (0.80)__. O método 2 atingiu uma __acurácia de (0.79)__, __F1-Score (0.78)__ e __AUC (0.85)__.


## Dataset

Disponível em: https://github.com/UCSD-AI4H/COVID-CT

O dataset utilizado neste trabalho [Zhao et al. 2020] possui 746 imagens de TC de Tórax em formato JPG e PNG, coletados entre o período de janeiro e abril de 2020, referentes à 760 _preprints_ encontrados em diferentes repositórios médicos. A coleta das imagens contidas nos preprints e todo o pré-processamento foi descrito em [Yang et al. 2020] e reflete o diagnóstico médico de 271 pacientes. Dentre as imagens, 349 (47%) são casos de COVID-19 (diagnóstico positivo) e 397 (53%) de casos normais (diagnóstico negativo).

As dimensões das imagens variam, na altura foi observado um mínimo de 153 pixels, uma
média de 491 pixels e um máximo de 1853 pixels. Já para o comprimento foi observado
um mínimo de 124 pixels, uma média de 383 pixels e um máximo de 1485 pixels. A
validade do _dataset) foi confirmada por um radiologista do Hospital Tingji, da cidade de Wuhan, China.

Os dados foram divididos em 3 subconjuntos, seguindo a Tabela 1. Por questões
comparativas, adotamos a divisão proposta em [He et al. 2020], ou seja, 57% para o
treinamento da CNN, 16% para validação e 27% para teste.
