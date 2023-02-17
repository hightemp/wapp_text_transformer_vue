<template>
    <div v-if="bShow">
        <div class="overlay"></div>
        <div class="modal show" tabindex="-1">
            <div class="modal-dialog">
                <div class="modal-content">
                    <div class="modal-header">
                        <h5 class="modal-title">{{title}}</h5>
                        <button type="button" class="btn-close" data-bs-dismiss="modal" aria-label="Close" @click="fnClose"></button>
                    </div>
                <div class="modal-body" v-html="sContentHTML"></div>
                <!-- <div class="modal-footer">
                    <button type="button" class="btn btn-secondary" data-bs-dismiss="modal" @click="fnClose">Close</button>
                </div> -->
                </div>
            </div>
        </div>
    </div>
</template>

<script>

import { mapMutations, mapState, mapActions, mapGetters } from 'vuex'
import { a, cc } from "../lib"

export default {
    props: ['window_name'],

    computed: {
        sContentHTML() {
            return this.$store.state.oWindow[this.window_name].content_html 
        },
        sTitle() {
            return this.$store.state.oWindow[this.window_name].title 
        },
        bShow() { 
            return this.$store.state.oWindow[this.window_name].window_show 
        }
    },

    methods: {
        ...mapMutations(a`fnHideWindow`),
        fnClose() {
            this.fnHideWindow(this.window_name)
        },
    }
}
</script>

<style>

</style>