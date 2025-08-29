# caseCirclesCloud

https://mayvic.github.io/organic-circle-cloud/

Uma nuvem de circulos organicos, que similar a nuvem de palavras demonstra a partir de tamanho relativo, a presença de maior ou menor importância em cada circulo.
Os circulos contem:
- valor total da aparição da palavra relacionada: define o tamanho dos circulos
- valor de aparições negativas: define a porção vermelha do anel
- valor de aparições neutras: define a porção amarela do anel
- valor de aparições positivas: define a porção verde do anel
- tooltip com as porcentagens de cada aparição: aparece acima de cada circulo ao ser sobreposto pelo cursor
- posicionamento organico: o circulo de maior tamanho fica posicionado ao centro e os demais se encaixam ao redor, visando não se sobrepor e nem sair do espaço visivel

## Technologies
Foi utilizado para o desenvolvimento Vue e NodeJs em suas últimas versões. 
- A escolha foi dada por familiaridade e demanda

TypeScript foi a linguagem utilizada.
- A escolha foi dada por familiaridade e necessidade de tipificação para maior clareza do código

Para o posicionamento organico foi utilziada a biblioteca d3-force.
- A escolha foi dada para possibilitar a abstração da física envolvida e simular uma melhor dinamicidade entre os circulos

## Challenges
### Posicionamento organico
Inicialmente considerei calcular a fisica para gerar um posicionamento menos linear, porém com pesquisa encontrei a biblioteca que simula a forma gravitacional e permite alinhar os demais circulos a partir de um circulo no centro

### Tooltip 
Enfrentei um desafio para que os tooltips ao mesmo tempo que se relacionassem com o circulo correspondente também ficassem sobrepostos aos demais circulos possibilitando a leitura. Para resolver minha solução foi criar os tooltips após a criacao de todos os circulos e gerar o CSS dinamico para cada par circulo-tooltip

### Grafico de Donut
Enfrentei o desafio de criar um circulo dividido em 3 cores percentuais utilizando SVG, após pesquisas encontrei como criar apenas um percentual do circulo, como solucao criei 3 camadas de circulos, sendo a vermelha a base, seguida pela amarela que possui seu proprio percentual mais o do verde, e por ultimo a camada verde que sobrepoe as demais. Resultando assim na visibilidade do respectivo percentual para cada cor.

## Future Improvements
- adicionar visualização por sentimento a partir de um filtro. Onde o circulo central seria o que possui mais ocorrencia do sentimento em questao. Para melhor diferenciacao visual, a porção correspondente do grafico de donut seria mais grossa que as demais que não estao em destaque.
- separacao das responsabilidades do codigo, criando o fluxo de chamada de endpoint e consumindo o arquivo no final do fluxo
- separacao de componentes, criando componentes especificos para circle e tooltip

## Recommended IDE Setup

[VSCode](https://code.visualstudio.com/) + [Volar](https://marketplace.visualstudio.com/items?itemName=Vue.volar) (and disable Vetur).

## Type Support for `.vue` Imports in TS

TypeScript cannot handle type information for `.vue` imports by default, so we replace the `tsc` CLI with `vue-tsc` for type checking. In editors, we need [Volar](https://marketplace.visualstudio.com/items?itemName=Vue.volar) to make the TypeScript language service aware of `.vue` types.

## Customize configuration

See [Vite Configuration Reference](https://vite.dev/config/).

## Project Setup

```sh
npm install
```

### Compile and Hot-Reload for Development

```sh
npm run dev
```

### Type-Check, Compile and Minify for Production

```sh
npm run build
```
