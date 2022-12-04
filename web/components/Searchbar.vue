<template>
    <form @submit.prevent="handleSearch()" class="center" :action="this.searchEngines[this.searchEngine].action" :method="this.searchEngines[this.searchEngine].method">
        <button v-if="this.usingNonDefaultEngine" class="searchbarPrefix" @click="buttonResetSearchEngine()">
            <img :src="`/images/search_engines_svg/${this.searchEngines[usedSearchEngine].icon}_Default.svg`" alt="" />
        </button>
        <button v-if="this.searching_shortcut" class="searchbarPrefix" @click="buttonResetSearchShortcut()">
            <img src="/images/other_icons_svg/shortcut_default.svg" alt="" />
        </button>
        <input v-model="form.search" ref="search" @focus="handleFocus()" @blur="handleBlur()" @input="handleInput()" @keydown.delete="keyboardResetSearchEngine($event)" :class="(this.usingNonDefaultEngine || this.searching_shortcut) ? 'searchbar withPrefix' : 'searchbar'" type="search" :name="this.searchEngines[this.usedSearchEngine].queryname" :placeholder="tempPlaceholder" autocomplete="off">
    </form>
</template>

<script>
    export default {
        props: [
            'name',
            'description',
            'searchEngine',
            'placeholder',
            'isDefault',
            'log',
            'smartSearch',
            'searchEngines',
            'shortcuts'
        ],
        mounted() {
            if (this.isDefault) {
                this.focusSearchBar();
            }
        },
        methods: {
            handleSearch() { 
                if (this.form.search == '') { return; }

                // [TODO] als search query == link of shortcut naam --> ga naar link of shortcut url in plaats van search query uit te voeren
                if (this.smartSearch){
                    this.handleSmartSearch();               
                } else if (this.searchEngines[this.usedSearchEngine].method.toLowerCase() == "get") {
                    // [TODO] log
                    this.doRedirect();
                } else {
                    // [TODO] zoek dees nog uit --> isda echt nodig?
                    console.log("post");
                }

                this.form.search = "";
            },

            handleSmartSearch() {
                // als het een URL is, ga naar de URL
                if (/^(https?:\/\/)?([-a-zA-Z0-9@:%._\+~#=]{2,256}\.[a-z]{2,6})\S*$/gm.test(this.form.search)) {
                    this.doRedirect(/^https?:\/\//gm.test(this.form.search) ? this.form.search : `http://${this.form.search}`);
                // als het een localhost link is, ga naar localhost
                } else if (/^(localhost):?(\d{1,4}|[1-5]\d{4}|6[0-4]\d{3}|65[0-4]\d{2}|655[0-2]\d|6553[0-5])?$/gm.test(this.form.search)) {
                    this.doRedirect("http://" + this.form.search)
                // als het ip is, ga naar het ip
                } else if (/^(?:(?:25[0-5]|2[0-4]\d|1?\d?\d)(?:(\.|:)(?!$)|$)){4}(\d{1,4}|[1-5]\d{4}|6[0-4]\d{3}|65[0-4]\d{2}|655[0-2]\d|6553[0-5])?$/gm.test(this.form.search)) {
                    this.doRedirect("http://" + this.form.search);
                // als het een shortcut is, ga naar de shortcut url
                } else if (/^#/gm.test(this.form.search)) {
                    for (var i = 0; i < this.shortcuts.length; i++) {
                        if (this.shortcuts[i].keywords) {
                            for (var j = 0; j < this.shortcuts[i].keywords.length; j++) {
                                const re = new RegExp(`^#(${this.shortcuts[i].keywords[j]})$`, 'gmi');

                                if (re.test(this.form.search)){
                                    this.doRedirect(this.shortcuts[i].url);
                                }
                            }
                        }
                    }
                    this.resetShortcutSearch();
                
                // Als geen van bovenstaande, ga naar searchengine
                } else if (this.searchEngines[this.usedSearchEngine].method.toLowerCase() == "get") {
                    // [TODO] log
                    this.doRedirect();
                } else {
                    // [TODO] zoek dees nog uit
                    console.log("post");
                }
                return;
            },

            doRedirect(url) {
                if (!url) { 
                    if (this.searchEngines[this.usedSearchEngine].split_query) {
                        // split query on '|'
                        let q = this.form.search.split(/ ?\| ?/i);

                        // replace ' ' with '+'
                        for (let arg in q) { q[arg] = q[arg].replace(/[ ,]+/g, '+'); }

                        // generate url
                        url = this.searchEngines[this.usedSearchEngine].link;
                        q.forEach((arg) => {
                            url = url.replace(/\{searchquery\}/i, arg);
                        })
                    } else {
                        url = this.searchEngines[this.usedSearchEngine].link.replace(/(.+)(\{searchquery\})(.*)/gmi, `$1${this.form.search}$3`);
                    }
                }
                window.location.href = url;
            },

            focusSearchBar() {
                this.$refs.search.focus();
            },

            handleFocus() {
                this.tempPlaceholder = `> ${this.placeholder}`;
                this.searchbarFocussed = true;
            },

            handleBlur() {
                this.tempPlaceholder = this.placeholder;
                this.searchbarFocussed = false;
            },

            handleInput() {
                if (this.smartSearch && /^@/gm.test(this.form.search)) {
                    for (var i = 0; i < this.searchEngines.length; i++) {
                        for (var j = 0; j < this.searchEngines[i].keywords.length; j++) {
                            const re = new RegExp(`^@(${this.searchEngines[i].keywords[j]}) `, 'gmi');

                            if (re.test(this.form.search)){
                                this.usedSearchEngine = i;
                                this.usingNonDefaultEngine = true;
                                this.form.search = '';
                            }
                        }
                    }
                } else if (this.smartSearch && /^#/gm.test(this.form.search)) {
                    this.searching_shortcut = true;
                } else if (this.smartSearch && !/^#/gm.test(this.form.search)) {
                    this.searching_shortcut = false;
                }
            },

            resetSearchEngine() {
                this.usedSearchEngine = this.searchEngine;
                this.usingNonDefaultEngine = false;
            },

            buttonResetSearchEngine() {
                if (this.smartSearch && !this.searchbarFocussed) {
                    this.resetSearchEngine();
                }
            },

            keyboardResetSearchEngine(e) {
                if (this.smartSearch && this.usingNonDefaultEngine && this.form.search == '' && e.key=='Delete') {
                    this.resetSearchEngine();
                }
            },

            resetShortcutSearch() {
                this.form.search = "";
                this.searching_shortcut = false;
            },

            buttonResetSearchShortcut() {
                if (this.smartSearch && !this.searchbarFocussed) {
                    this.resetSearchEngine()
                } 
            }
        },
        data() {
            return {
                hover_searchEngine: false,
                usedSearchEngine: this.searchEngine,
                usingNonDefaultEngine: false,
                searching_shortcut: false,
                form: {
                    search: ""
                },
                tempPlaceholder: this.placeholder,
                searchbarFocussed: false
            }
        }
    }
</script>