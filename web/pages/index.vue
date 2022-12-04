<template>
  <div class="center">
    <h1 id="title" class="title">._.</h1>

    <div class="searchbars center">
      <Searchbar v-for="i in searchbars" v-bind:key="i.name" :description="i.description" :searchEngine="defaultSearchEngine" :placeholder="i.placeholder" :isDefault="i.isDefault" :log="i.log" :smartSearch="i.smartSearch" :searchEngines="searchengines" :shortcuts="shortcuts" />
    </div>

    <div class="shortcuts center">
      <Shortcut v-for="i in shortcuts" v-bind:key="i.name" v-if="!i.hidden" :name="i.name" :description="i.description" :url="i.url" :type="i.type" :icon="i.icon" :log="i.log" />
    </div>

    <p class="version">V2.06</p>

  </div>
</template>

<script>
import searchbarData from "~/assets/configfiles/searchbars.json";
import shortcutData from "~/assets/configfiles/shortcuts.json";
import searchengineData from "~/assets/configfiles/searchengines.json"
// [TODO] hover component --> description 

export default {
  name: 'IndexPage',

  data () {
    return {
      // [TODO] get data from file of database --> pak misschien maar gwn file voor dit
      searchbars: searchbarData,
      shortcuts: shortcutData,
      searchengines: searchengineData,
      defaultSearchEngine: 0
    }
  }
}
</script>
