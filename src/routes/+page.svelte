<h1>Welcome to SvelteKit</h1>
<p>Visit <a href="https://kit.svelte.dev">kit.svelte.dev</a> to read the documentation</p>

<script lang="ts">
    import { census } from "./census.js";

    let letters = Array(25);
    let loadletters = "A,B,D,E,E,F,G,H,I,J,D,D,X,S,K,D,E,D,C,D,D,S,E,S,D";
    let values = [];
    let cacheWords = undefined;
    for (let index = 0; index < letters.length; index++) {
        letters[index] = "";
    }
    let points = 0;

    let calcPoints = () => {
        points = 0;
        Object.keys(census).forEach(k => {
            if(check(k.toUpperCase()))
                points += census[k];
        });
    }
        

    let check = (checkWord) => {
        console.log("Check is called", checkWord);
        values = [];
        let checkLength = checkWord.length;
        if (checkWord == "" || checkWord == null)
            return;
        cacheWords = new Array(checkLength);
        for (let i = 0; i < checkLength; i++)
        {
            cacheWords[i] = new Array(25); 
            for (let j = 0; j < 25; j++) {
                cacheWords[i][j] = new Set();
            }
        }
        var indexes = [], i = -1;
        while ((i = letters.indexOf(checkWord[0], i+1)) != -1){
            indexes.push(i);
            cacheWords[checkLength - 1][i] = new Set([letters[i]]);
        }
        nextmoves(checkLength - 1, checkWord);
        // console.log(cacheWords);

        values = cacheWords[0].filter(a => a.size > 0).map(a => Array.from(a).join(" ")).join("\n");

        // console.log(cacheWords, values);

        return cacheWords[0].some(a => a.size > 0);
    }

    let kingsmoves = [
        [1,0], [0,1], [1,1], [-1,0], [0,-1], [-1,1], [-1,-1], [1,-1], 
    ]; // kings moves in (dx, dy)


    let nextmoves = (count, checkWord) => {
        if (count == 0) return;

        for (let y = 0; y < 25; y++) {
            var coord = [Math.floor(y/5), y%5];
            
            for (let i = 0; i < kingsmoves.length; i++) {
                let move = kingsmoves[i];
                let nextX = coord[0] + move[0];
                let nextY = coord[1] + move[1];
                
                
                if (nextX >= 5 || nextY >= 5 || nextX < 0 || nextY < 0)
                    continue;
            
                
                var prevIndex = coord[0] * 5 + coord[1] % 5;
                var nextIndex = nextX * 5 + nextY % 5;
                var currLetter = letters[nextIndex];

                if (prevIndex > 25 || nextIndex > 25)
                    console.log(coord, nextX, nextY);
    
                
                if (cacheWords[count][prevIndex] == null)
                    cacheWords[count][prevIndex] = new Set();
    
                if (cacheWords[count - 1][nextIndex] == null)
                    cacheWords[count - 1][nextIndex] = new Set();
    
                var newWords = Array.from(cacheWords[count][prevIndex]).map(a => a + currLetter);
                // console.log("filter", checkWord, newWords);
                newWords = newWords.filter(a => checkWord.startsWith(a));
                cacheWords[count - 1][nextIndex] = cacheWords[count - 1][nextIndex].union(new Set(newWords));

            }
        }
        nextmoves(count - 1, checkWord);
    }
</script>
<small>Load letters:</small>
<input bind:value={loadletters} maxlength="50">
<button on:click={() => letters = (loadletters + " ").split(',').map(l => l.trim())}>Load</button>
<div style="display: block;height:2em"></div>
<!-- <small>Check word:</small>
<input bind:value={checkWord}> -->
<button on:click={() => calcPoints()}>Check</button>
<div style="display: block;height:2em"></div>
<div>
    {#each Array.from(letters.keys()) as row (row)}
        <input class="one-letter" bind:value={letters[row]} maxlength="1" oninput="this.value = this.value.toUpperCase()">
        {#if row % 5 === 4}
            <br/>
        {/if}
    {/each}
</div>
<div>
    <small>{letters}</small>
    <small>Points: {points}</small>
    <small>Values: {values}</small>
</div>

<style>
    input.one-letter {
        height:30px;
        width:30px;
        text-align: center;
        margin: 5px;
    }
</style>
