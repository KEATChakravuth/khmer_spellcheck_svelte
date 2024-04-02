<script>
    import { onMount } from "svelte";
    import { Content, SpellCheck} from "../../stores"

    let content = ''; // State to manage the content
    
    async function fetchData(){
      try {
        const API_ENDPOINT = "";
        SpellCheck.set("");
        const res = await fetch(API_ENDPOINT, {
          method: 'POST',
          headers:{
            'Content-Type': 'application/json'
          },
          body: JSON.stringify({message: content})
        });
        const data = await res.json();
        Content.set(data.input);
        SpellCheck.set(data.output);
      } catch (error) {
        console.error("Error fetching data", error);
      }
    }
    onMount(() => {
      const observer = new MutationObserver(() => {
        fetchData();
      });
      const targetNode = document.querySelector('[contenteditable=true]');
      observer.observe(targetNode, { childList: true, subtree: true });
    });

</script>

<h3 style="text-align: center;">Type Here</h3>
<div contenteditable = true class="text" bind:textContent={content}>{content}</div>
<br>
<h3 style="text-align: center;">SpellCheck</h3>
<div class="correction">{$SpellCheck}</div>

<style>
    /* .highlight {
      color: red;
    } */
    .text{
        border: none;
        margin: 1rem 0rem;
        border-radius: 0.5rem;
        font-size: 1.5rem;
        padding: 0.5rem;
        max-height: 30vh;
        min-height: 30vh;
        overflow-y: auto;
        background-color: rgb(241, 241, 241);
    }
    .text:focus{
        outline: none;
    }
    .text:empty::before{
        content: attr();
        color: rgb(150, 150, 150);
    }
    .correction{
        border: none;
        margin: 1rem 0rem;
        border-radius: 0.5rem;
        font-size: 1.5rem;
        padding: 0.5rem;
        overflow-y: auto;
        background-color: rgb(241, 241, 241);
    }
</style>