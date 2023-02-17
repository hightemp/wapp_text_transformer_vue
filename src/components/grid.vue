<template>
    <div class="table-actions-panel">
        <div class="spacer"></div>
        <button class="btn btn-light" @click="fnFirst"><i class="bi bi-chevron-bar-left"></i></button>
        <button class="btn btn-light" @click="fnPrevShift"><i class="bi bi-chevron-double-left"></i></button>
        <button class="btn btn-light" @click="fnPrev"><i class="bi bi-chevron-compact-left"></i></button>
        <div class="input-group mb-3">
            <input type="text" class="form-control" v-model="sPage" />
            <span class="input-group-text" id="basic-addon1">/ {{iMaxPages}}</span>
        </div>
        <button class="btn btn-light" @click="fnNext"><i class="bi bi-chevron-compact-right"></i></button>
        <button class="btn btn-light" @click="fnNextShift"><i class="bi bi-chevron-double-right"></i></button>
        <button class="btn btn-light" @click="fnLast"><i class="bi bi-chevron-bar-right"></i></button>

        <button class="btn btn-light" @click="fnAddClick"><i class="bi bi-plus-square"></i></button>
        <button class="btn btn-light" @click="fnEditClick"><i class="bi bi-pencil"></i></button>
        <button class="btn btn-light" @click="fnRemoveClick"><i class="bi bi-trash"></i></button>

        <div class="spacer"></div>
    </div>
    <div class="table">
        <div class="row" :style="{display:'grid', 'grid-template-columns': '1fr '.repeat(iStructLength) }">
            <div v-for="(oSF, sK) in oStruct" :key="sK" class="cell header">
                {{oSF.label}}
                <input type="text" class="form-control" @input="(oE) => fnFilterInput(oE, sK)" :value="oTable.filter[sK]"  />
            </div>
        </div>
        <template v-for="oRow in aSlicedRows" :key="oRow">
            <div 
                :class="'row '+(oSelectedItem && oSelectedItem.id == oRow.id ? 'active' : '')" 
                :style="{display:'grid', 'grid-template-columns': '1fr '.repeat(iStructLength) }" 
                @click="(oE) => fnItemClick(oRow)"
            >
                <div v-for="(oSF, sK) in oStruct" :key="sK" class="cell">{{oRow[sK]}}</div>
            </div>
        </template>
    </div>
    <edit_window title="Редактирование" :form_name="sFormName" :table_name="table_name" />
</template>

<script>
import { mapMutations, mapState, mapActions, mapGetters } from 'vuex'
import { a, cc } from "../lib"

import edit_window from "./edit_window.vue"

export default {
    props: ['table_name'],

    components: {
        edit_window,
    },

    computed: {
        iStructLength() {
            return Object.keys(this.oStruct).length
        },
        oStruct() {
            return this.$store.state.oStructure[this.table_name]
        },
        oTable() {
            return this.$store.state.oDatabase[this.table_name]
        },
        iMaxPages() {
            return Math.ceil(this.aRows.length / this.iPageCount)
        },
        aRows() {
            var aRows = this.oTable.data.filter((oI) => {
                var bResult = true;
                for (var sK in this.oTable.filter) {
                    console.log(this.oTable.filter[sK])
                    if (this.oTable.filter[sK]) {
                        console.log(this.oTable.filter[sK])
                        bResult = bResult && ~oI[sK].indexOf(this.oTable.filter[sK])
                    }
                }
                return bResult
            })

            return aRows;
        },
        aSlicedRows() {
            var aRows = this.aRows.slice((this.sPage-1)*this.iPageCount, this.sPage*this.iPageCount)
            return aRows;
        },
    },

    data() {
        return {
            sPage: 1,
            iPageCount: 10,

            sFormName: 'table2',

            oSelectedItem: null,
        }
    },

    methods: {
        ...mapMutations(a`fnShowEditWindow fnRemoveFromTable`),
        fnFilterInput(oE, sK) {
            this.$store.commit('fnUpdateFilter', { sTableName: this.table_name, sName: sK, sV:oE.target.value })
        },
        fnFirst() {
            this.sPage = 1
        },
        fnPrevShift() {
            if (this.sPage>5) {
                this.sPage-=5
            }
        },
        fnPrev() {
            if (this.sPage>1) {
                this.sPage-=1
            }
        },
        fnLast() {
            this.sPage = this.iMaxPages
        },
        fnNextShift() {
            if (this.sPage<this.iMaxPages-5) {
                this.sPage+=5
            }
        },
        fnNext() {
            if (this.sPage<this.iMaxPages) {
                this.sPage+=1
            }
        },
        fnAddClick() {
            this.fnShowEditWindow({ sFormName: this.sFormName, oItem: {} })
        },
        fnEditClick() {
            if (this.oSelectedItem) {
                this.fnShowEditWindow({ sFormName: this.sFormName, oItem: this.oSelectedItem })
            } else {
                alert("Нужно выбрать")
            }
        },
        fnRemoveClick() {
            if (this.oSelectedItem) {
                this.fnRemoveFromTable({ sTableName: this.table_name, oItem: this.oSelectedItem })
            } else {
                alert("Нужно выбрать")
            }
        },
        fnItemClick(oRow) {
            this.oSelectedItem = oRow
        }
    }
}
</script>

<style>
.table-actions-panel {
    display: flex;
    height: 29px;
}
.table-actions-panel > .input-group {
    width: 100px;
}
.table-actions-panel > .spacer {
    flex: 1;
}
.table {
    width: 100%;
}
.row.active {
    background: blue;
    color: white;
}
.row:first-child {
    border-top: 1px solid #eee;
}
.cell:first-child {
    border-left: 1px solid #eee;
}
.cell {
    padding: 4px;
    border-right: 1px solid #eee;
    border-bottom: 1px solid #eee;
}
.cell.header {
    background: #eee;
    font-weight: bold;
}
</style>