<script>
    import {Content, SpellCheck} from "../stores"

    let currentIndex = 0;
    let suggestions;
    let selectedSuggestions = [];
    let spellcheck_copy;
    let isInputSelected = []; // Track whether input is selected for each suggestion

    let count;

    function parseOptions(str) {
        // For example: str = "i['go', 'went'] toschool in ['morning', 'afternoon', 'evening']"
        // output [['go', 'went'], ['morning', 'afternoon', 'evening']]
        const regex = /\[([^\]]+)\]/g;
        const matches = str.matchAll(regex);
        const result = [];
        
        for (const match of matches) {
            const options = match[1].split(',');
            result.push(options.map(option => option.trim()));
        }
        
        return result;
    }

    // Function to handle event when "Next" button is clicked
    // function onNext() {
    //     currentIndex++; // Increment the index to show the next item
    //     if (currentIndex >= suggestions.length) {
    //     currentIndex = 0; // Reset to the first item when reaching the end
    //     }
    // }

    let results = [];
    const unsubscribe = SpellCheck.subscribe(value => {
        currentIndex = 0;
        // Log the value of the SpellCheck store
        // create a copy(a cheat) to not effect the original spellcheck
        spellcheck_copy = value;

        suggestions = parseOptions(value);
        console.log(suggestions);
        // Count suggestions
        count = 0;
        for (let i = 0; i < suggestions.length; i++) {
            count += suggestions[i].length;
        }

        // Create list of corrected statement before incorrected word
        results = [];
        let currentSubstring = '';
        let insideBrackets = false;
        for (let i = 0; i < value.length; i++) {
            if (value[i] === '[') {
                insideBrackets = true;
                // Push the currentSubstring into results if it's not empty
                if (currentSubstring.trim() !== '') {
                    results.push([currentSubstring]);
                }
                currentSubstring = '';
            } else if (value[i] === ']') {
                insideBrackets = false;
                // Push the currentSubstring into results if it's not empty
                if (currentSubstring.trim() !== '') {
                    results.push([currentSubstring]);
                }
                currentSubstring = '';
            } else if (!insideBrackets) {
                currentSubstring += value[i];
            }
        }
        // Push the currentSubstring into results if it's not empty at the end of the loop
        if (currentSubstring.trim() !== '') {
            results.push([currentSubstring]);
        }

        // Discard the last item from the results array
        results.pop();

        // Initialize isInputSelected array
        isInputSelected = Array(suggestions.length).fill(false);
    });

    const onChange = (e, i) => {
        const selectedWord = e.target.value;
        selectedSuggestions[i] = selectedWord;
        isInputSelected[i] = true; // Set to true when input is selected
    }
    
    const handleCorrectTextAccept = (index) => {
        // let original = $Content;
        let spellcheck = $SpellCheck;

        // Regular expression to match text within brackets including brackets
        let regex = /\[([^\]]+)\]/g;

        // Array to store all matches
        let matches = [];
        let match;

        // Find all matches
        while (match = regex.exec(spellcheck_copy)) {
            matches.push(match[1]);
        }

        // Extract the value from the first set of brackets
        let BracketValue = matches.length >= index+1 ? matches[index] : null;

        // Extracting the correct word based on the index
        let selected = selectedSuggestions[index];
        let correctedWord = selected ? selected : '';
        
        // Replace the word in the original text
        let correctedText = spellcheck.replace('[' + BracketValue + ']', correctedWord);
        
        // Update
        handleCorrectTextDismiss(index);
        // if(suggestions.length == 0) {
        //     unsubscribe();
        // }
        SpellCheck.set(correctedText);
    }

    const handleCorrectTextDismiss = (index) => {
        
        // Extracting the index
        
        // Removing the item at the extracted index
        suggestions.splice(index, 1);
        suggestions = [...suggestions];

        // Updating count
        count = 0;
        for (let i = 0; i < suggestions.length; i++) {
            count += suggestions[i].length;
        }
        
        // this part <p style="margin: 8px;">{results[i]+"___"}</p>
        results.splice(parseInt(index), 1);
        results = [...results];

        // Regular expression to match text within brackets including brackets
        let regex = /\[([^\]]+)\]/g;

        // Array to store all matches
        let matches = [];
        let match;

        // Find all matches
        while (match = regex.exec(spellcheck_copy)) {
            matches.push(match[1]);
        }

        // Extract the value from the first set of brackets
        let BracketValue = matches.length >= index+1 ? matches[index] : null;
        
        spellcheck_copy = spellcheck_copy.replace('[' + BracketValue + ']', BracketValue);
        // onNext()
    }
</script>

<div class="suggestion">
    <h3 class="title">
        <span class="count">{count}
        </span> 
        {#if count <= 1}
            Suggestion
        {:else}
            Suggestions
        {/if} 
    </h3>
    <button class="btn-red">Check correctness</button>
    <button class="btn-blue">Check synonyms</button>
    <button class="btn-green">Translate to Your Language</button>
    <div class="active">
        <div class="wrapper">
            {#if suggestions.length > 0 && suggestions[currentIndex]}
                <div class="correct-text">
                    <div class="correct-text-header">
                        <svg class="header-icon" xmlns="http://www.w3.org/2000/svg" height="1em" viewBox="0 0 512 512"><path d="M256 0c4.6 0 9.2 1 13.4 2.9L457.7 82.8c22 9.3 38.4 31 38.3 57.2c-.5 99.2-41.3 280.7-213.6 363.2c-16.7 8-36.1 8-52.8 0C57.3 420.7 16.5 239.2 16 140c-.1-26.2 16.3-47.9 38.3-57.2L242.7 2.9C246.8 1 251.4 0 256 0zm0 66.8V444.8C394 378 431.1 230.1 432 141.4L256 66.8l0 0z"/></svg> 
                        <p>Correction</p>
                    </div>
                    <div class="word-select">
                        <p style="margin: 8px;">{results[currentIndex]+"___"}</p>
                        {#each suggestions[currentIndex] as suggestion, i}
                            {#if suggestions[currentIndex].length > 1}
                                <input type="radio" id={i + suggestion} name="suggestion" value={suggestion} on:change={(e) => onChange(e, currentIndex)}/>
                                <label for={i + suggestion}>{suggestion}</label>
                            {:else}
                                <input type="radio" id={i + suggestion} name="suggestion" value={suggestion} on:click={(e) => onChange(e, currentIndex)}/>
                                <label for={i + suggestion}>{suggestion}</label>
                            {/if}                            
                        {/each}
                    </div>
                    <div class="correct-text-footer">
                        <button class="btn-accept" on:click={() => handleCorrectTextAccept(currentIndex)} disabled={!isInputSelected[currentIndex]}>Accept</button>
                        <button class="btn-dismiss" on:click={() => handleCorrectTextDismiss(currentIndex)}>Dismiss</button>
                        <!-- <button class="btn-dismiss" on:click={() => handleCorrectTextDismiss(currentIndex)}>Next</button> -->
                    </div>
                </div>
            {:else}
                <!-- <img class="active-img" src="correct.png" alt="correct" /> -->
                <svg class="active-img" fill="none" viewBox="0 0 64 65" role="img" aria-hidden="true" focusable="false"><g clip-path="url(#EmptyE_a)"><path fill="#fff" stroke="#0E101A" stroke-linecap="round" stroke-linejoin="round" stroke-miterlimit="10" d="M47.87 26.1c-.06 0-.1-.05-.1-.1.09-3.4-1.08-4.63-4.47-4.73-.06 0-.1-.05-.1-.1 0-.06.05-.1.1-.1 3.4.09 4.63-1.08 4.73-4.47 0-.06.05-.1.1-.1.06 0 .1.05.1.1-.09 3.4 1.08 4.63 4.47 4.72.06 0 .1.05.1.1 0 .06-.05.1-.1.1-3.4-.09-4.63 1.08-4.73 4.47 0 .07-.05.11-.1.11ZM10.68 28.5c-.06 0-.1-.04-.1-.1.09-3.11-.98-4.24-4.1-4.33-.04 0-.08-.04-.08-.1 0-.05.04-.08.1-.08 3.11.08 4.24-.99 4.33-4.1 0-.05.04-.09.1-.09.05 0 .08.04.08.1-.08 3.11.99 4.24 4.1 4.32.05 0 .09.04.09.1 0 .05-.04.08-.1.08-3.11-.08-4.24.99-4.33 4.1 0 .06-.04.1-.1.1ZM35.84 12.5c-.06 0-.1-.06-.1-.12.1-3.96-1.27-5.4-5.23-5.51-.07 0-.11-.06-.11-.13 0-.06.06-.1.12-.1 3.96.1 5.4-1.27 5.51-5.23 0-.07.06-.11.13-.11.06 0 .1.06.1.12-.1 3.97 1.27 5.4 5.23 5.5.07 0 .11.06.11.13 0 .06-.06.1-.12.1-3.96-.1-5.4 1.27-5.51 5.23-.01.08-.06.12-.13.12Z"></path><path fill="#0E101A" stroke="#0E101A" stroke-linecap="round" stroke-linejoin="round" stroke-miterlimit="10" d="M2.19 6.8a2.53 2.53 0 1 1 2.74-4.24l30.2 19.54-2.74 4.24L2.19 6.81Z"></path><path fill="#fff" stroke="#0E101A" stroke-linecap="round" stroke-linejoin="round" stroke-miterlimit="10" d="M24.3 21.1a2.52 2.52 0 1 1 2.74-4.23l8.75 5.65-2.74 4.24-8.75-5.66Z"></path><circle cx="34.41" cy="24.67" r="2.55" fill="#fff" stroke="#0E101A" stroke-linecap="round" stroke-linejoin="round" stroke-miterlimit="10" transform="rotate(-57 34.4 24.67)"></circle><path fill="#C6CBDE" d="M32 64.5c10.6 0 19.2-1.61 19.2-3.6s-8.6-3.6-19.2-3.6-19.2 1.61-19.2 3.6 8.6 3.6 19.2 3.6Z"></path><path fill="#6E66DE" stroke="#0E101A" stroke-linecap="round" stroke-linejoin="round" stroke-miterlimit="10" d="M42.84 34.42H21.16l-3.23 23.34c-.2 1.49.5 2.9 1.94 3.3 2.13.6 6 1.27 12.62 1.27 6.77 0 10.35-.7 12.17-1.3 1.17-.4 1.64-1.56 1.47-2.78l-3.29-23.83Z"></path><path fill="#F8C6DA" stroke="#0E101A" stroke-linecap="round" stroke-linejoin="round" stroke-miterlimit="10" d="m43.74 41.35-.9-6.93H21.16l-.9 6.93h23.48Z"></path><path fill="#6E66DE" stroke="#0E101A" stroke-linecap="round" stroke-linejoin="round" stroke-miterlimit="10" d="M50.08 33.3H13.92a1.12 1.12 0 0 0 0 2.24h36.16a1.12 1.12 0 1 0 0-2.24Z"></path></g><defs><clipPath id="EmptyE_a"><path fill="#fff" d="M0 .5h64v64H0z"></path></clipPath></defs></svg>
                <h5 class="active-message">Start writing to make the magic happen.</h5>
                <p style="text-align: center;">Suggestions will appear here.</p>
            {/if}
        </div>
    </div>
</div>

<style>
    .count {
        display: inline-block;
        width: 30px; /* Adjust size as needed */
        height: 30px; /* Adjust size as needed */
        background-color: #646B81; /* Background color */
        color: white; /* Text color */
        border-radius: 50%; /* Makes it circular */
        text-align: center;
        line-height: 30px; /* Vertically centers the text */
        font-size: 16px; /* Adjust font size as needed */
    }
    .active{
        width: 100%;
        display: flex;
        flex-direction: column;
        justify-content: center;
        /* max-height: 100%; Set the maximum height */
    }
    .active-img{
        width: 50%;
        padding: 1rem;
        display: block;
        margin: 6rem auto;

    }
    .active-message{
        font-size: 1rem;
        font-weight: bold;
        text-align: center;
    }
    .suggestion {
        width: 35%;
        padding: 1rem;
        /* border: 1px solid #ccc; Add a border for clarity */
        /* border-radius: 0.5rem; */
    }
    .title{
        font-size: 1rem;
        font-weight: bold;
    }
    .btn-red{
        padding: 0.5rem;
        background-color: rgb(231, 27, 27);
        border: none;
        color: white;
        border-radius: 0.5rem;
        margin: 0.2rem 0.5rem 0 0;
    }
    .btn-blue{
        padding: 0.5rem;
        background-color: rgb(3, 190, 223);
        border: none;
        color: white;
        border-radius: 0.5rem;
        margin: 0.2rem 0.5rem 0 0;
    }
    .btn-green{
        padding: 0.5rem;
        background-color: rgb(21, 224, 14);
        border: none;
        color: white;
        border-radius: 0.5rem;
        margin: 0.2rem 0.5rem 0 0;
    }
    .wrapper{
        margin-top: 16px;
        margin-bottom: 16px;
        max-height: 1000px;
        overflow: auto;
        scroll-behavior: smooth;
    }
    .correct-text{
        -webkit-box-shadow: 0px 0px 6px 1px rgb(207, 207, 207);
        -moz-box-shadow: 0px 0px 6px 1px rgb(207, 207, 207);
        box-shadow: 0px 0px 6px 1px rgb(207, 207, 207);
        padding: 1rem;
        margin: 1rem 0;
        border-radius: 0.5rem;
        margin-right: 8px;
        margin-left: 8px;
        
        border: 1px solid #ccc; /* Add a border for clarity */
        border-radius: 0.5rem;
    }
    .correct-text-header{
        color: grey;
        display: flex;

    }
    .header-icon{
        width: 20px;
        height: 20px;
        fill: rgb(231, 27, 27);
        vertical-align: middle;
        margin: 0.1rem 0.3rem;
    }
    .correct-text-footer{
        margin: 1rem 0 0 0;
    }
    .btn-accept {
        padding: 0.5rem 1rem; /* Increase padding for better button appearance */
        background-color: rgb(42, 182, 84);
        border: none;
        color: white;
        border-radius: 0.5rem;
        margin: 0 0.5rem 0 0;
        cursor: pointer;
        transition: background-color 0.3s, box-shadow 0.3s; /* Smooth transition effect */

        /* Add box shadow for depth */
        box-shadow: 0 2px 4px rgba(0, 0, 0, 0.1); /* Adjust shadow as needed */
    }

    /* Hover effect */
    .btn-accept:hover {
        background-color: rgb(32, 162, 74); /* Darken the background color on hover */
        box-shadow: 0 4px 8px rgba(0, 0, 0, 0.1); /* Increase shadow on hover for depth effect */
    }

    /* Active (clicked) state effect */
    .btn-accept:active {
        transform: translateY(1px); /* Push the button down slightly when clicked */
    }
    .btn-accept:disabled{
        background-color: #cccccc;
        color: #333;
        cursor:auto;
    }

    .btn-accept:disabled:hover{
        transform:scale(1);
        opacity: 1;
    }

    .btn-dismiss {
        padding: 0.5rem 1rem; /* Increase padding for better button appearance */
        background-color: white;
        border: none;
        color: gray;
        border-radius: 0.5rem;
        margin: 0 0.5rem 0 0;
        cursor: pointer;
        transition: background-color 0.3s, box-shadow 0.3s; /* Smooth transition effect */

        /* Add box shadow for depth */
        box-shadow: 0 2px 4px rgba(0, 0, 0, 0.1); /* Adjust shadow as needed */
    }

    /* Hover effect */
    .btn-dismiss:hover {
        background-color: lightgrey; /* Darken the background color on hover */
        box-shadow: 0 4px 8px rgba(0, 0, 0, 0.1); /* Increase shadow on hover for depth effect */
    }

    /* Active (clicked) state effect */
    .btn-dismiss:active {
        transform: translateY(1px); /* Push the button down slightly when clicked */
    }

    .word-select{
        margin-top: 20px;
        margin-bottom: 20px;
        font-size: 1.3rem;
    }
    /* Hide the default radio button */
    .word-select input[type="radio"] {
        display: none;
    }

    /* Style the custom radio button */
    .word-select input[type="radio"] + label {
        display: inline-block;
        padding: 5px 10px;
        border: 1px solid #ccc;
        border-radius: 5px;
        cursor: pointer;
    }

    /* Style for checked state */
    .word-select input[type="radio"]:checked + label {
        background-color: #007bff;
        color: #fff;
        border-color: #007bff;
    }
</style>