<template>
    <div class="dropdown">
        <a 
            class="btn btn-light dropdown-toggle" 
            href="#" 
            role="button" 
            data-bs-toggle="dropdown" aria-expanded="false" 
            @click="fnMenuOpen"
        >
        </a>

        <ul :class="'dropdown-menu '+(bOpened ? 'show' : '')">
            <li v-for="oI in aItems" :key="oI.id"><a class="dropdown-item" href="#" @click="fnOnItemClick(oI)" v-html="oI.title"></a></li>
        </ul>
    </div>
</template>

<script>
export default {
    props: ['items'],

    emits: ['clickitem'],

    data() {
        return {
            aItems: this.items,
            bOpened: false
        }
    },
    methods: {
        fnFocusOut() {
            this.bOpened = false
        },
        fnMenuOpen() {
            this.bOpened = !this.bOpened
        },
        fnOnItemClick(oI) {
            this.bOpened = false
            this.$emit('clickitem', oI)
        }
    },
    created() {
        window.addEventListener('click', (e) => {
            if (!this.$el.contains(e.target)){
                this.bOpened = false
            }
        })        
    }
}
</script>

<style>

</style>