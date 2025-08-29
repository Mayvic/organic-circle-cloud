<script setup lang="ts">
//recebe o arquivo data
import data from "./assets/sentiment_data_json.json"
import { forceSimulation, forceCenter, forceCollide } from "d3-force";
import { ref, onMounted, onUnmounted } from "vue";

// define altura e largura
let width = 900;
let height = 700;

//ordena do maior para o menor
let dataSorted = data.sort((n1,n2) => {
    if (n1.total > n2.total) {
        return -1;
    }

    if (n1.total < n2.total) {
        return 1;
    }

    return 0;
});

//cria a classe Circle
class Circle{
    maxR: number;
    minR: number;
    circumference: number;
    maxValue: number;
    minValue: number;
    scaleValue: number;
    r: number;
    total: number;
    positivo: number;
    neutro: number;
    negativo: number;
    percentPositivo: number;
    percentNeutro: number;
    percentNegativo: number;
    normalizedWord: string;
    fx?: number;
    fy?: number;
    x: number;
    y: number;


    constructor(maxValue:number, minValue: number, dataItem:any){
        //define valores máximo e mínimo de raio
        this.maxR = 100;
        this.minR = 50;

        this.x = 0;
        this.y = 0;

        //recebe o maior valor entre os dados e o menor
        this.maxValue = maxValue;
        this.minValue = minValue;
       
        this.total = dataItem.total ? dataItem.total :0;
        //coloca o total de respostas para esse item na escala entre todas as respostas
        this.scaleValue = (this.total - this.minValue)/(this.maxValue - this.minValue)
        //calcula o raio do circulo de acordo com a escala definida
        this.r = (this.scaleValue*(this.maxR-this.minR)) + this.minR;
        //calcula a circunferencia de acordo com o raio definido
        this.circumference = 2* 3.14 * this.r; 
        this.positivo = dataItem.positivo  ? dataItem.positivo :0;
        this.neutro = dataItem.neutro  ? dataItem.neutro :0;
        this.negativo = dataItem.negativo  ? dataItem.negativo :0;
        this.percentPositivo = dataItem.percentPositivo  ? dataItem.percentPositivo :0;
        this.percentNeutro = dataItem.percentNeutro  ? dataItem.percentNeutro :0;
        this.percentNegativo = dataItem.percentNegativo  ? dataItem.percentNegativo :0;
        this.normalizedWord = dataItem.normalizedWord  ? dataItem.normalizedWord :0;

        //utiliza a escala e o percentual positivo como sementes para gerar posições "aleatórias" de inicio
        this.x = this.scaleValue * ((0.80*width) - (0.20*width)) + (0.20*width)
        this.y = this.percentPositivo/100 * ((0.80*height) - (0.20*height)) + (0.20*height)
    
    }
}


//verifica se os circulos saem da fronteira definida
function checkBounds(circle: Circle){
    let raiusWithMargin = circle.r + 30 + 120;
  if ((circle.x - raiusWithMargin) < 0) circle.x = raiusWithMargin;
  if ((circle.x + raiusWithMargin) > width) circle.x = width - raiusWithMargin;
  if ((circle.y - raiusWithMargin) < 0) circle.y = raiusWithMargin;
  if ((circle.y + raiusWithMargin) > width) circle.y = height - raiusWithMargin;
}

//
let c:Circle[] = [];
let circleArray = ref(c);
let simulation: any;

onMounted(() => {
    //cria os circulos e inicializa seus valores
    circleArray.value = createCircles(dataSorted);
    //cria o css e relaciona cada circulo com seu tooltip
    createStyle(dataSorted);
    
    let circleSorted = circleArray.value;
    //centraliza o maior circulo no meio
    circleSorted[0].fx = width/2; //centraliza em x
    circleSorted[0].fy = height/2; //centraliza em y

    simulation = forceSimulation(circleSorted)
        .force("center", forceCenter(width/2 , height/2))
        //12 é a margem entre os circulos
        // 0.5 é a força de repulsa ao colidir
        .force("collide", forceCollide((c :Circle) => c.r + 12).strength(0.5))
        .alphaDecay(0.02) // controla velocidade de animação
        .on("tick", () => {
            //valida a fronteira
            for(const c of circleSorted) checkBounds(c);
            // Atualiza círculos a cada iteração
            circleArray.value = circleSorted.map(d => ({ ...d }));
        });
});

onUnmounted(() => {
    //finaliza a simulacao
    if (simulation) simulation.stop();
});

function createCircles(dataSorted: any[]): Circle[]{
    //como os itens estão ordenados, a maior quantidade de respostas 
    //esta no primeiro item do array e a menor no ultimo
    let maxValue = dataSorted[0].total;
    let minValue = dataSorted[dataSorted.length - 1].total;
    let circleArray: Circle[] = [];
    for(const c of dataSorted){
        //cria as instancias e adiciona no vetor
        const itemCircle = new Circle(maxValue, minValue, c);
        circleArray.push(itemCircle)
    }
    return circleArray
}

function createStyle(dataSorted: any[]){
    let styles = "";
    //itera sobre todos os circulos
    for(const i in dataSorted){
        //cria a logica de mostrar/esconder para cada par circulo-tooltip
        styles+= `
        .groupCircle${i} ~ .tooltip${i}{
            visibility: hidden;
        }
        .groupCircle${i}:hover ~ .tooltip${i}{
            visibility: visible;
        }
        `
    }
    //injeta no dom
    const styleEl = document.createElement("style");
    styleEl.textContent = styles;
    document.head.appendChild(styleEl);
}

</script>

<template>
    <svg  
        :width="`${width}`" 
        :height="`${height}`" 
        :viewBox="`0 0 ${width} ${height}`"
    >
        <!-- sombra -->
        <defs> 
            <filter id="f1">
            <feDropShadow dx="0" dy="0" stdDeviation="5" flood-opacity="0.2"/>
            </filter>
        </defs> 
        <!-- renderiza os circulos -->
        <g v-for="(circleItem, index) in circleArray" 
            :transform="`translate(${circleItem.x},${circleItem.y})`"
            :class="`groupCircle${index}`"
        >
            <!-- circulo de fundo branco -->
            <circle :r="`calc(${circleItem.r + 10})`" fill="white" filter="url(#f1)"/>
            <!-- palavra -->
             <text :cx="`calc(${circleItem.x}/2)`" :cy="`calc(${circleItem.y}/2)`" text-anchor="middle" dominant-baseline="middle" fill="black">
                {{ circleItem.normalizedWord }}
            </text>
            <!-- circulo negativo - ocupa 100% -->
            <circle :r="`${circleItem.r}`" fill="transparent" 
                stroke="#F44336"
                stroke-width="10" 
            />  
            <!-- circulo neutro - ocupa (neutro+positivo)% -->
            <circle :r="`${circleItem.r}`" fill="transparent" 
                stroke="#FFC107"
                stroke-width="10"
                :stroke-dasharray="`calc(${circleItem.percentPositivo + circleItem.percentNeutro} * ${circleItem.circumference} / 100) ${circleItem.circumference}`" 
                :transform="`rotate(-90) `" 
            /> 
            <!-- circulo positivo - ocupa positivo% -->
            <circle  :r="`${circleItem.r}`" fill="transparent" 
                stroke="#4CAF50"
                stroke-width="10"
                :stroke-dasharray="`calc(${circleItem.percentPositivo} * ${circleItem.circumference} / 100) ${circleItem.circumference}`" 
                :transform="`rotate(-90)`" 
            /> 
        </g>  
        <!-- tooltip -->
        <g v-for="(circleItem, index) in circleArray" 
            :transform="`translate(${circleItem.x -60},${circleItem.y - circleItem.r - 120})`"
            :class="`tooltip${index}`"
            opacity="0.7">
        >
             <rect rx="5" width="120" height="100" fill="black" opacity="0.9s"></rect>
            
             <text x="15" y="25" fill="white">{{circleItem.normalizedWord}}</text>
             <text x="15" y="45" fill="white">Positivo: {{ Math.round(circleItem.percentPositivo) }}%</text>
             <text x="15" y="65" fill="white">Neutro: {{ Math.round(circleItem.percentNeutro) }}%</text>
             <text x="15" y="85" fill="white" >Negativo: {{ Math.round(circleItem.percentNegativo) }}%</text>
             <polygon points="55,100 65,100 60,105" fill="black"></polygon>
         </g> 
    </svg>
    
</template>

<style scoped>

</style>
