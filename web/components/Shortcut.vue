<template>
    <button class="shortcut" @click="clicked()" @mouseover="hover = true" @mouseleave="hover = false">
        <p>{{name}}</p>
        <img :src="hover ? `/images/shortcuts_svg/${icon}_Hover.svg` : `/images/shortcuts_svg/${icon}_Default.svg`" alt="" />
    </button>
</template>

<script>
    // [TODO] icons

    export default {
        props: [
            'name',
            'description',
            'type',
            'url',
            'icon',
            'log'
        ],
        methods: {
            clicked() {
                if (this.type == 'external'){
                    // [TODO] log
                    window.location.href = this.url;
                } else {
                    this.$router.push(this.url);
                }
            },
        },

        data () {
            return {
                hover: false
            }
        }
    }
</script>