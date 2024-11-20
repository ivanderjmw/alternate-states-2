<h1 class="text-4xl font-bold">janestreet: alternate-states-2</h1>

<script lang="ts">
    import Button from "$lib/components/ui/button/button.svelte";
    import Input from "$lib/components/ui/input/input.svelte";
    import { census } from "./census.js";

    let letters = Array(25);
    let loadletters = "W,Y,I,H,A,G,H,O,D,J,D,D,X,S,K,D,E,D,C,D,D,S,E,S,D";
    let values = [];
    let cacheWords = undefined;
    for (let index = 0; index < letters.length; index++) {
        letters[index] = "";
    }
    let points = 0;

    let calcPoints = () => {
        points = 0;
        values = [];
        Object.keys(census).forEach(k => {
            if(check(k.toUpperCase()))
            {
                points += census[k];
                values.push(k);
            }
        });
    }

    let check = (checkWord) => {
        console.log("Check is called", checkWord);
        let checkLength = checkWord.length;
        if (checkWord == "" || checkWord == null)
            return;
        cacheWords = new Array(checkLength);
        for (let i = 0; i < checkLength; i++)
        {
            cacheWords[i] = new Array(25); 
            for (let j = 0; j < 25; j++) {
                cacheWords[i][j] = i == checkLength - 1 ? new Set([letters[j]]) : new Set();
            }
        }
        nextmoves(checkLength - 1, checkWord);

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
                newWords = newWords.filter(a => isOneLetterDiff(checkWord,a));
                cacheWords[count - 1][nextIndex] = cacheWords[count - 1][nextIndex].union(new Set(newWords));

            }
        }
        nextmoves(count - 1, checkWord);
    }

    let isOneLetterDiff = (a, b) => {
        let diff = 0;
        for (let i = 0; i < Math.min(a.length, b.length); i++) {
            if (a[i] != b[i])
                diff++;
        }
        return diff <= 1;
    }
</script>
<div style="display: block;height:2em"></div>
<div class="mx-auto flex items-center justify-center gap-2">
<small>Load letters:</small>
<Input bind:value={loadletters} class="max-w-32" />
<Button on:click={() => letters = (loadletters + " ").split(',').map(l => l.trim())}>Load</Button>
<Button on:click={() => letters = ("1,".repeat(24) + "1").split(',').map(l => l.trim())}>Clear</Button>
<Button on:click={() => calcPoints()}>Check</Button>
</div>
<div style="display: block;height:2em"></div>

<div class="grid grid-cols-1 md:grid-cols-2 w-max mx-auto">
<div>
    <div class="mx-auto w-max grid gap-2 grid-cols-5 text-center justify-center place-items-center">
        {#each Array.from(letters.keys()) as row (row)}
            <input class="rounded h-16 w-16 bg-slate-300 text-center text-3xl font-extrabold" bind:value={letters[row]} maxlength="1" oninput="this.value = this.value.toUpperCase()">
        {/each}
    </div>
    <div class="flex flex-col">
        <small>{letters}</small>
        <small>Points: {points}</small>
        <small>Values: {values.join(' ')}</small>
    </div>
</div>
<div class="mx-auto w-max">
    <table class="table-auto">
        <thead>
        <tr class="bg-slate-200"><th>City</th><th>Points</th></tr>
    </thead>
    <tbody>
        {#each Object.keys(census).sort((a, b) => values.some(v => v == b) - values.some(v => v == a)) as key (key)}
        <tr style="background-color:{values.some(v => v == key) ? 'yellow' : 'red'}">
            <td class="text-center w-32" >{key.toUpperCase()}</td>
            <td class="text-center w-32" >{census[key]}</td></tr>
        {/each}
    </tbody>
    </table>
</div>
</div>