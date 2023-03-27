<script>
  import './tailwind.css'

  let wordLengths = [4, 5, 6, 7, 8, 9];
  let wordLength = 5;
  let wordSrc = "API";
  let word = "";
  let properWords = [];
  let tries = [];
  let keyboardLetters = [["q","w","e","r","t","y","u","i","o","p"],["a","s","d","f","g","h","j","k","l"],["ENTER","z","x","c","v","b","n","m","cancel"]]
  let hitLetters = [];
  let sunkLetters = [];
  let missedLetters = [];
  let files;
  let darkMode = false;
  let contrastMode = false;
  let wordChecking = true;
  let rows = [];

  async function getWordFromApi(){
    const res = await fetch('https://random-word-api.herokuapp.com/word?length='+wordLength);
    let data = await res.json();
    return data[0];
  }

  async function getProperWords(){
    const res = await fetch('https://random-word-api.herokuapp.com/all');
    return res.json();
  }

  async function startGame() {
    tries = []
    hitLetters = []
    sunkLetters = []
    missedLetters = []
    rows = []
    properWords = await getProperWords();
    for(let i = 0; i <= wordLength; i++)
      rows.push(i)
    rows = rows
    switch(wordSrc){
      case "API":
        word = await getWordFromApi();
        break;
      case "JSON":
        if(!files){
          alert("Choose JSON file.")
          return
        }
        let reader = new FileReader()
        reader.readAsText(files[0])
        reader.onload = function(){
          // @ts-ignore
          let words = JSON.parse(reader.result)[wordLength]
          word = words[Math.floor(Math.random()*words.length)]
        }
        break;
    }
  }

  document.body.addEventListener("keydown", (e)=>{
    if(word == "")
      return
    if(e.keyCode >= 65 && e.keyCode <= 90){
      addLetter(e.key)
    }
    else if(e.keyCode == 8){
      deleteLetter();
    }
    else if(e.keyCode == 13){
      approveTry();
    }
  })

  function addLetter(letter){
    if(tries.length != 0 && tries[tries.length -1].length < wordLength + 1){
      let lastTry = tries[tries.length - 1]
      if(lastTry.length == wordLength)
        return
      // Add letter to last try
      tries[tries.length - 1] += letter
    }
    else{
      // Add new try
      tries.push(letter)
    }
    tries = tries
  }

  function deleteLetter(){
    if(tries.length != 0 && tries[tries.length - 1].length > 0 && tries[tries.length - 1][tries[tries.length - 1].length - 1] != "."){
      tries[tries.length - 1] = tries[tries.length - 1].slice(0, -1);
    }
  }

  function approveTry(){
    if(tries.length != 0 && tries[tries.length - 1].length == wordLength){
      if(wordSrc == "API" && wordChecking && !properWords.includes(tries[tries.length - 1])){
        alert("Not in word list!")
        return
      }
      let thisTry = tries[tries.length - 1]
      for(let i = 0; i < wordLength; i++){
        let letter = thisTry[i]
        if(!sunkLetters.includes(letter)){
          let hitsAllSpots = false;
          for(let j = 0; j < wordLength; j++){
            if(word[j] == letter){
              hitsAllSpots = false
              for(let k = 0; k < tries.length; k++){
                if(tries[k][j] == letter){
                  hitsAllSpots = true
                  break
                }
              }
            }
          }
          if(hitsAllSpots && letter == word[i]){
            if(hitLetters.includes(letter)){
              hitLetters.splice(hitLetters.indexOf(letter))
            }
            sunkLetters.push(thisTry[i])
          }
          else if(word.includes(letter)){
            if(!hitLetters.includes(letter)){
              hitLetters.push(letter);
            }
          }
          else{
            if(!missedLetters.includes(letter)){
              missedLetters.push(letter);
            }
          }
        }
      }
      keyboardLetters = keyboardLetters
      tries[tries.length - 1] += "."
      if(thisTry == word){
        alert("Congratulations! You won in " + tries.length + " move(s).")
        word = ""
      }
      else if(tries.length == rows.length){
        alert("No more tries! The word was " + word.toUpperCase() + ".")
        word = ""
      }
    }
  }

  function letterProperlyHit(tryIndex, letterIndex){
    let currentTry = tries[tryIndex]
    let currentLetter = currentTry[letterIndex]
    let demand = 0
    let supply = 0
    for(let i = 0; i < word.length; i++){
      if(i <= letterIndex && currentTry[i] == currentLetter && word[i] != currentLetter)
        demand += 1
      if(word[i] == currentLetter && currentTry[i] != currentLetter)
        supply += 1
    }
    return supply >= demand
  }

  function showHint(){
    let hint = ""
    for(let i = 0; i < Math.ceil(wordLength/2); i++){
      hint += word[i]
    }
    if(tries.length > 0 && !tries[tries.length - 1].includes("."))
      tries[tries.length - 1] = hint
    else
      tries.push(hint)
    while(rows.length > tries.length)
      rows.pop()
    tries = tries
    rows = rows
  }
</script>

<div class="flex flex-col items-center w-full h-screen" class:darkMode="{darkMode}">
  {#if word == ""}
    <div>
      <select bind:value={wordLength} class:darkMode="{darkMode}">
        {#each wordLengths as i}
          <option value="{i}">{i}</option>
        {/each}
      </select>
      |
      <select bind:value={wordSrc} class:darkMode="{darkMode}">
        <option value="API">API</option>
        <option value="JSON">JSON</option>
      </select>
      |
      {#if wordSrc == "JSON"}
        <label for="jsonFile">Select JSON file:</label>
        <input type="file" id="jsonFile" name="jsonFile" accept="application/json" bind:files={files}>
      {:else}
        <label for="checkingCb">Check words</label>
        <input type="checkbox" name="checkingCb" id="checkingCb" bind:checked="{wordChecking}">
      {/if}
      |
      <label for="themeCb">Dark mode</label>
      <input type="checkbox" name="themeCb" id="themeCb" bind:checked="{darkMode}">
      |
      <label for="contrastCb">Contrast mode</label>
      <input type="checkbox" name="contrastCb" id="contrastCb" bind:checked="{contrastMode}">
    </div>
    <button class="border-solid border-2 p-3 pt-1 pb-1 m-2" class:darkMode="{darkMode}" on:click={startGame}>Start</button>
  {:else}
    {#await properWords then properWords}
      <div class="flex flex-col justify-center">
        {#each rows as i}
            <div class="flex flex-row justify-center">
              {#if tries.length > i}
                {#each tries[i].replace(".", "") as j, index}
                  <div class="border-solid border-2 h-14 w-14 m-1 items-center justify-center flex text-xl"
                    class:bg-lime-600="{(tries[i][tries[i].length - 1] == "." && word[index] == j && !contrastMode)}"
                    class:bg-yellow-400="{(tries[i][tries[i].length - 1] == "." && word[index] != j && letterProperlyHit(i, index) && !contrastMode)}"
                    class:bg-gray-500="{(tries[i][tries[i].length - 1] == "." && (!word.includes(j) || !letterProperlyHit(i, index)))}"
                    class:bg-orange-600="{(tries[i][tries[i].length - 1] == "." && word[index] == j && contrastMode)}"
                    class:bg-blue-400="{(tries[i][tries[i].length - 1] == "." && word[index] != j && letterProperlyHit(i, index) && contrastMode)}"
                    class:darkMode="{darkMode}">
                    {j.toUpperCase()}
                  </div>
                {/each}
                {#each Array(wordLength - tries[i].replace(".", "").length) as j}
                  <div class="border-solid border-2 h-14 w-14 m-1 items-center justify-center flex text-xl" class:darkMode="{darkMode}"></div>
                {/each}
              {:else}
                {#each word as j}
                  <div class="border-solid border-2 h-14 w-14 m-1 items-center justify-center flex text-xl" class:darkMode="{darkMode}"></div>
                {/each}
              {/if}
            </div>    
        {/each}
      </div>
      <br>
      <button class="flex flex-col justify-center m-5 p-2 border-solid border-2" on:click={showHint}>Use hint</button>
      <div>
        {#each keyboardLetters as keyboardRow}
          <div class="flex flex-row justify-center">
            {#each keyboardRow as letter}
              {#if letter.length == 1}
                <button on:click={()=>{addLetter(letter)}} class="border-solid border-2 h-14 w-10 m-1 items-center justify-center flex text-xl" 
                  class:bg-lime-600="{sunkLetters.includes(letter) && !contrastMode}"
                  class:bg-yellow-400="{hitLetters.includes(letter) && !contrastMode}"
                  class:bg-gray-500="{missedLetters.includes(letter)}"
                  class:bg-orange-600="{sunkLetters.includes(letter) && contrastMode}"
                  class:bg-blue-400="{hitLetters.includes(letter) && contrastMode}"
                  class:darkMode="{darkMode}">   
                  {letter.toUpperCase()}
                </button>
              {:else if letter == "ENTER"}
                <button on:click={approveTry} class="border-solid border-2 h-14 w-20 m-1 items-center justify-center flex text-xl" class:darkMode="{darkMode}">ENTER</button>
              {:else}
                <button on:click={deleteLetter} class="border-solid border-2 h-14 w-20 m-1 p-4 items-center justify-center flex text-xl" class:darkMode="{darkMode}">
                  <img src="{darkMode ? 'src/assets/alternateBackspace.png' : 'src/assets/backspace.png'}" alt="backspace">
                </button>
              {/if}
            {/each}
          </div>
        {/each}
      </div>
    {/await}
  {/if}
</div>