
<script>
    import { onMount } from "svelte";
    import { Content, SpellCheck} from "../../stores"
    import { API_ENDPOINT } from '../../../env';

    let content = ''; // State to manage the content
    
    async function fetchData(){
      try {
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
  
<div
  contenteditable = true
  class="text"
  bind:textContent={content}
>
  {content}
</div>

<div
  class="text"
>
  {$SpellCheck}
</div>
  
<style>
    .text{
        border: none;
        margin: 1rem 0rem;
        border-radius: 0.5rem;
        font-size: 1.5rem;
        padding: 0.5rem;
        max-height: 30vh;
        min-height: 30vh;
        overflow-y: auto;
    }
    .text:focus{
        outline: none;
    }
    .text:empty::before{
        content: attr();
        color: rgb(150, 150, 150);
    }
</style>