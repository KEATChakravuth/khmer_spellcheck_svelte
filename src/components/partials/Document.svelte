<script>
    import TextCustomBar from "./partials/TextCustomBar.svelte";
    import ContentEditable from "./partials/ContentEditable.svelte";
</script>

<div class="document">
    <input type="text" class="title" placeholder="Untitled Document">
    <TextCustomBar/>
    <ContentEditable/>
</div>

<style>
    .document{
        width: 45%;
        display: flex;
        flex-direction: column;
    }
    .title{
        border: none;
        border-radius: 0.5rem;
        font-weight: bold;
        width: 50%;
        padding: 0.5rem;
        margin: 0rem 0rem 5rem 0;
    }
    .title:focus{
        outline: none;
        background-color: rgb(232, 228, 247);
    }

</style>