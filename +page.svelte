

<script lang="ts">

import { onMount } from "svelte";
// import { Stream } from "stream";
import { elapsed,testjson, ip } from '$src/stores/store1';
// import { ip } from "$src/stores/store1";

// import brain from "brain.js"
let brain
  let errors = 0;
  let forecast, trainingdata, predictresult
  let errorthresh = 0.09
  let iteration = 1000000

onMount(async() => {

//   const w = (window as any);
// w.process = {
//   env: { DEBUG: undefined },
// };
// w.Buffer = [];
// w.process = { env: { DEBUG: undefined }, version: [] };
// w.global = w;

brain = await import('brain.js');

//     const LSTM = brain.recurrent.LSTM;
// const net = new LSTM();
// // used to build list below
// // const mathProblemsSet = new Set();
// // for (let i = 0; i < 10; i++) {
// //   for (let j = 0; j < 10; j++) {
// //     mathProblemsSet.add(`${i}+${j}=${i + j}`);
// //     mathProblemsSet.add(`${j}+${i}=${i + j}`);
// //   }
// // }
// // const mathProblems = Array.from(mathProblemsSet);
// // console.log(mathProblems);

// const mathProblems = [
  
// "12","13","12","13","12","13","12","13","12","13","12","13","12","13","12","13","12","13","12","13","12","13","12","13","12","13","12","13","12","13"
// ];

// net.train(mathProblems, { iterations: 1600, log: true, errorThresh: 0.03, timeout:2 });

// errors = 0;
// // if (elapsed<duration) {
  
//   for (let i = 0; i < mathProblems.length; i++) {
//   // const input = mathProblems[i].split('=')[0] + '=';
//   const input = mathProblems[i];
//   const output = net.run(input);
//   const predictedMathProblem = input + output;
//   const error = mathProblems.indexOf(predictedMathProblem) <= -1 ? 1 : 0;
//   errors += error;
//   console.log(input + output + (error ? ' - error' : ''));
// }
// console.log('Errors: ' + errors / mathProblems.length);
predictit()

})

var t = {
  tr: {
    "This page has been open for": "Bu sayfa şu kadar süredir açık",
    "seconds": "saniye",
    "Training data": "Öğrenme verisi",
    "Forecast": "Tahminler",
    "Predict": "Tahmin et",
    "JSON is not valid": "JSON geçersiz",
    "JSON is valid": "JSON geçerli",
    "Error": "Hata payı",
    "Iterations": "Tekrarlama",
    "Providing the numbers above, it is going to predict more on below": "Yukarıdaki sayılardan, aşağıda sonraki sayılar tahmin edilecektir",
  },
};

// Same test as previous, but combined on a single set
trainingdata = JSON.stringify([
  [
    [1, 5],
    [2, 4],
    [3, 3],
    [4, 2],
    [5, 1],
  ],
]);

const predictit = () => {
  
const net = new brain.recurrent.LSTMTimeStep({
  inputSize: 2,
  hiddenLayers: [10],
  outputSize: 2,
  
});

// console.log(trainingdata, "ehehe", trainingdata)
// iterations: 100000,
predictresult = net.train(JSON.parse(trainingdata), { iterations: iteration,log: true, errorThresh: errorthresh });

// const closeToFiveAndOne = net.run([
//   [1, 5],
//   [2, 4],
//   [3, 3],
//   [4, 2],
// ]);
const closeToFiveAndOne = net.run(JSON.parse(trainingdata)?.[0].slice(0,trainingdata.length-1));

// console.log(closeToFiveAndOne);

// now we're cookin' with gas!
forecast = net.forecast(
  [
    JSON.parse(trainingdata)?.[0]?.[0],
    JSON.parse(trainingdata)?.[0]?.[1],
  //     [1, 5],
  // [2, 4],
  ],
  3
);

console.log('next 3 predictions', forecast);
}

// $: forecast2 = forecast?.map(x => JSON.stringify(`${x[0]}: ${x[1]}`, undefined, 4))

  
</script>

<!-- {performance.now()} -->
<div class="twitter-card">
<p>
	
  {ip("This page has been open for", t)}
	{$elapsed} {$elapsed === 1 ? ip("second", t) : ip("seconds", t)}
</p>
<p>{ip("Providing the numbers above, it is going to predict more on below", t)}
</p>

<p>{ip("Training data", t)} </p>

<textarea cols="40" rows="10" bind:value={trainingdata}></textarea>

{#if !testjson(trainingdata)?.[0]}


  <p>{ip("JSON is not valid", t)} ❌</p>
  <p>	{testjson(trainingdata)}</p>
{:else}
<p>{ip("JSON is valid", t)} ✅   </p>
	



<p>{ip("Error", t)}: {#if predictresult}
  {predictresult?.["error"]} 
  {/if} </p>

<textarea cols="40" rows="1" bind:value={errorthresh}></textarea>

<p>{ip("Iterations", t)}: {#if predictresult}
  {predictresult?.["iterations"]}  
 {/if} </p>

<textarea cols="40" rows="1" bind:value={iteration}></textarea>

<p><button on:click={() => predictit()}>{ip("Predict", t)} </button></p>
{/if}

<p>{ip("Forecast", t)}</p>
{#if forecast?.length>0}
{#each forecast as it}
<p>{it}</p>
{/each}
{/if}


</div>
