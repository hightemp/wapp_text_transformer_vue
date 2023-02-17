<template>
  <div class="wrapper">
    <div class="left-panel">
      <button v-for="(oMenuItem, iI) in aMenu" :key="iI" class="btn" @click="fnClickLeftMenu(oMenuItem)" :title="oMenuItem.title"><i :class="'bi '+oMenuItem.icon"></i></button>
      <hr />
      <button class="btn" title="Экспортировать" @click="fnExport"><i class="bi bi-box-arrow-down"></i></button>
      <button class="btn btn-import" title="Импортировать"><i class="bi bi-box-arrow-in-up"></i><label><input type="file" ref="file_selector" @change="fnFileImportChange" /></label></button>
    </div>
    <template v-if="sMode=='files'">
      <div class="files-panel">
        <list 
          :table_name="'files'" 
          :form_name="'files'" 
          :filter_field="'name'" 
          :default_item="oDefaultFileItem">
          <template v-slot:default="p">
            {{ p.oItem.name }}
          </template>
        </list>
      </div>
      <template v-if="oSelectedFile">
        <div class="text-panel">
          <div class="text-block">
            <div class="text-block-left-panel">
              <list 
                :table_name="'files_texts'" 
                :form_name="'files_texts'" 
                :filter_field="'name'" 
                :default_item="oDefaultFileTextsItem">
                <template v-slot:default="p">
                  {{ p.oItem.name }}
                </template>
              </list>
            </div>
            <div class="text-block-editor-panel">
              <template v-if="oCurrentText">
                <v-ace-editor
                  v-model:value="sCurrentTextContent"
                  lang="text"
                  theme="chrome"
                  style="height: 100%" 
                />
              </template>
            </div>
          </div>
          <div class="text-action-panel">
            <div class="btn btn-light" @click="fnRunScript">Запустить скрипт</div>
            <div class="btn btn-light" @click="fnRunScriptAndAdd">Запустить и ответ в новой вкладке</div>
            <div class="btn btn-light" @click="fnShowHelp">Помощь</div>
          </div>
          <div class="scripts-block">
            <div class="scripts-left-panel">
              <list 
                :table_name="'scripts'" 
                :form_name="'scripts'" 
                :filter_field="'name'" 
                :default_item="oDefaultScriptItem">
                <template v-slot:default="p">
                  {{ p.oItem.name }}
                </template>
              </list>
            </div>
            <div class="scripts-editor-panel">
              <template v-if="oCurrentScript">
                <v-ace-editor
                  v-model:value="sCurrentScriptContent"
                  lang="javascript"
                  theme="chrome"
                  style="height: 100%" 
                />
              </template>
            </div>
          </div>
        </div>
      </template>
    </template>
    <template v-if="sMode=='scripts'">
      <div class="scripts-panel">
      </div>
    </template>
  </div>

  <info_window :window_name="'help'"/>
  <repo_window />
  <saved_toast />

  <loader/>
</template>

<script>

import ace from 'ace-builds';

import modeJsUrl from 'ace-builds/src-noconflict/mode-javascript?url';
ace.config.setModuleUrl('ace/mode/javascript', modeJsUrl);

import themeChromeUrl from 'ace-builds/src-noconflict/theme-chrome?url';
ace.config.setModuleUrl('ace/theme/chrome', themeChromeUrl);

import { VAceEditor } from 'vue3-ace-editor';

import list from "./components/list.vue"

import info_window from "./components/window.vue"
import repo_window from "./components/repo_window.vue"
import saved_toast from "./components/saved_toast.vue"
import loader from './components/loader.vue'

import { mapMutations, mapState, mapActions, mapGetters } from 'vuex'
import { a, cc } from "./lib"

export default {
  name: 'App',
  components: {
    list,
    info_window,
    repo_window,
    saved_toast,
    loader,
    VAceEditor
  },

  computed: {
    ...cc(`bShowRepoWindow bShowSaveToast`),
    oCurrentScript: {
      get() { return this.$store.state.oDatabase['scripts'][`selection_item`] }
    },
    sCurrentScriptContent: {
      get() { return this.oCurrentScript[`content`] },
      set(mV) { this.oCurrentScript[`content`] = mV },
    },
    oCurrentText() { return this.$store.state.oDatabase[`files_texts`][`selection_item`] },
    sCurrentTextContent: {
      get() { return this.oCurrentText[`text`] },
      set(mV) { this.oCurrentText[`text`] = mV },
    },
    aTexts: {
      get() { return this.$store.state.oDatabase[`files_texts`]['data'].filter((oI) => oI.file_id == this.oSelectedFile.id) },
    },
    oSelectedFile() { return this.$store.state.oDatabase['files'][`selection_item`]},
    oSelectedScript() { return this.$store.state.oDatabase['scripts'][`selection_item`]},

    oDefaultFileTextsItem() {
      return {
        name: "",
        text: "",
        file_id: this.oSelectedFile.id
      }
    }
  },

  data() {
    return {
      oDefaultFileItem: {
        name: "",
        texts: [
          { id: 1, name: "Текст", content: "", }
        ],
        selected_text: 0,
      },
      oDefaultScriptItem: {
        name: "",
        content: "",
      },
      sMode: "files",

      aMenu: [
        { id: "repo-window", title: "Выбрать репозиторий", icon: "bi-person-fill" },
        { id: "save", title: "Сохранить", icon: "bi-arrow-up-square" },
        { id: "files", title: "Файлы", icon: "bi-files" },
        { id: "scripts", title: "Скрипты", icon: "bi-filetype-js" },
      ],
    }
  },

  methods: {
    ...mapMutations(a`fnLoadRepos fnShowWindow fnAddTableItem`),
    ...mapActions(a`fnSaveDatabase fnImportDatabase fnExportDatabase`),
    fnClickLeftMenu(oItem) {
      if (oItem.id == "repo-window") {
        this.bShowRepoWindow = true
      }
      if (oItem.id == "save") {
        this.fnSaveAll()
      }

      if (oItem.id == "files") {
        this.sMode = "files"
      }
      if (oItem.id == "scripts") {
        this.sMode = "scripts"
      }
    },

    fnSaveAll() {
      this.fnSaveDatabase()
      this.bShowSaveToast = true
    },

    fnExport() {
      this.fnExportDatabase()
    },
    fnImport() {
      let oFile = this.$refs.file_selector.files[0];
      let reader = new FileReader();
      var oThis = this

      reader.readAsText(oFile);

      reader.onload = function() {
        oThis.fnImportDatabase(reader.result)
      };

      reader.onerror = function() {
        console.error(reader.error);
      };
    },
    fnFileImportChange() {
      this.fnImport()
    },
    fnShowHelp() {
      this.fnShowWindow('help')
    },
    fnRunScript() {
      var sText = this.sCurrentTextContent
      eval(`${this.sCurrentScriptContent}`)
      this.sCurrentTextContent = sText
    },
    fnRunScriptAndAdd() {
      var sText = ""+this.sCurrentTextContent
      eval(`${this.sCurrentScriptContent}`)
      var oItem = {
        name: this.oCurrentText.name+"*",
        text: sText,
      }
      this.fnAddTableItem({ sTableName: 'files_texts', oItem })
    },
  },
  created() {
    var oThis = this

    this.fnLoadRepos()

    document.addEventListener('keydown', e => {
      if (e.ctrlKey && e.keyCode === 83) {
          e.preventDefault();
          oThis.fnSaveAll()
      }
    });

    window.addEventListener('click', (e) => {
      if (this.bShowTaskMenu) {
        if (oThis.$refs.task_menu.contains(e.target)) {

        } else {
          this.bShowTaskMenu = false
        }
      }
    })        
  },
}
</script>

<style>
</style>
