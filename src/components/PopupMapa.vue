<template>
    <div class="feature-pop-up" >
        <h3>{{name}}</h3>
        <span>{{ detail }}</span>
    </div>
</template>

<script>

export default {
    name: 'PopupMapa',
    props: {
        name: String,
        detail: String,
    }
}
</script>

<style>
.feature-pop-up {
    border-image: url(../assets/border.png) 10 !important;
    font-size: 12px;
    background: white;
    padding: 5px;
    border-radius: 8px;
    border-style:  solid;
    border-width: 5px;
}

h3 {
    margin-top: 2px;
    margin-bottom: 3px;
}

</style>