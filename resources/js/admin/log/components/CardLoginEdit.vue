<template>
    <div>
        <v-card>
            <v-card-title>
                <v-icon>list</v-icon> {{ translate('card.edit_log')}}  {{ " " }}                  
                <v-btn class="ma5" @click="del()"  :disabled="!valid" color="lighten" dark>{{ translate('common.delete')}}</v-btn>

            </v-card-title>
            <v-divider class="mb-2"></v-divider>
            <v-form v-model="valid" ref="cardFormEdit" lazy-validation>
                <v-container grid-list-md>
                <v-layout row wrap>
                    <v-flex xs12 sm12>
                        <v-text-field v-bind:label="translate('common.key')" v-model="key" :rules="keyRules"></v-text-field>
                    </v-flex>
                    <v-flex xs12 sm8>
                        <v-select
                                v-bind:label="translate('common.select_branches')"
                                v-bind:items="options.branches"
                                v-model="branch"
                                item-text="name"
                                item-value="id"
                        ></v-select>
                    </v-flex>
                    <v-flex xs12 sm8>
                        <v-menu
                            ref="menu"
                            v-model="menu"
                            :close-on-content-click="false"
                            transition="scale-transition"
                            offset-y
                            min-width="auto"
                            >
                                <template v-slot:activator="{ on, attrs }">
                                    <v-text-field
                                        v-model="date"
                                        v-bind:label="translate('common.login_date')"
                                        prepend-icon="mdi-calendar"
                                        readonly
                                        v-bind="attrs"
                                        v-on="on"
                                    ></v-text-field>
                                </template>
                            <v-date-picker
                                ref="picker"
                                v-model="date"
                                :max="new Date().toISOString().substr(0, 10)"
                            >
                            </v-date-picker>
                            <v-col
                                cols="11"
                                sm="5"
                                >
                                    <v-menu
                                        ref="menu"
                                        v-model="menu2"
                                        :close-on-content-click="false"
                                        :nudge-right="40"
                                        :return-value.sync="time"
                                        transition="scale-transition"
                                        offset-y
                                        max-width="290px"
                                        min-width="290px"
                                    >
                                        <template v-slot:activator="{ on, attrs }">
                                        <v-text-field
                                            v-model="time"
                                            v-bind:label="translate('common.time')"
                                            prepend-icon="mdi-clock-time-four-outline"
                                            readonly
                                            v-bind="attrs"
                                            v-on="on"
                                        ></v-text-field>
                                        </template>
                                        <v-time-picker
                                        v-if="menu2"
                                        v-model="time"
                                        full-width
                                        @click:minute="$refs.menu.save(time)"
                                        ></v-time-picker>
                                    </v-menu>
                            </v-col>
                        
                    </v-menu>
                    </v-flex>
                    <v-flex xs12>
                        <v-btn @click="save()" :disabled="!valid" color="lighten" dark>{{ translate('common.update')}}</v-btn>
                    </v-flex>
                </v-layout>
            </v-container>
            </v-form>
        </v-card>
    </div>
</template>

<script>
    export default {
        props: {
            propId: {
                required: false
            },
        },
        data() {

            const self = this;

            return {
                id: '',
                menu2: false,
                modal2: false,
                valid: false,
                key: '',
                id: '',
                keyRules: [
                    (v) => !!v || this.translate('common.key_is_required'),
                ],
                date : '',
                time : '',
                branch : '',
                selectedBranch:null,
                branches: [],
                active:'',
                options: {
                    users: [],
                    branches: [],
                },
                alert: {
                    show: false,
                    icon: '',
                    color: '',
                    message: ''
                },
                branches_totalItems:0,
                pagination:{
                    branches_page:0,
                    branches_itemsPerPage:0,
                    branches_totalItems:0,
                }
            }
        },
        computed:{
            dateCT: function () {
                return this.date;
            }, 
            timeCT: function () {
                return this.time;
            }, 
        },
        mounted() {
            console.log('components.cardLoginEdit.vue');

            const self = this;

            this.id = this.$attrs.id != undefined ? this.$attrs.id : this.propId;

            this.onDelete = this.$attrs.on_delete;

            this.loadBranches(()=>{

            });

            this.loadCardLogin(()=>{

            });

            this.$store.commit('setBreadcrumbs',[
                {label:this.translate('card.title'),to:{name:'cards.list'}},
                {label:this.translate('card.edit_log'),to:''},
                ]);

        },
        methods: {
            save(){
                const self = this;

                let payload = {
                    key: self.key,
                    updated_at: self.date + " " +  self.time,
                    created_at: self.date + " " +  self.time,
                    branch_id: self.branch,
                };
                console.log(payload);

                self.$store.commit('showLoader');

                axios.put('/admin/cardlogin/'+self.id, payload).then(function(response) {
                    self.$store.commit('showSnackbar',{
                       message: response.data.message,
                       color: 'success',
                       duration: 3000
                    });

                    self.$eventBus.$emit('CARDLOGIN_UPDATED');
                    self.$store.commit('hideLoader');

                    // reset
                }).catch(function (error) {

                    self.$store.commit('hideLoader');

                    if (error.response) {
                        self.$store.commit('showSnackbar',{
                            message: error.response.data.message,
                            color: 'error',
                            duration: 3000
                        });
                    } else if (error.request) {
                        console.log(error.request);
                    } else {
                        console.log('Error', error.message);
                    }
                });
            },
            del(cb){
                const self = this;


                axios.delete('/admin/cardlogin/'+self.id).then((res) => {
                    self.$eventBus.$emit('CARDLOGIN_DELETED');
                    ( self.onDelete('log_edit',null) || Function() )

                });
            },
             loadBranches(cb) {

                const self = this;

                let params = {
                    page: self.pagination.branches_page,
                    per_page: self.pagination.branches_itemsPerPage
                };

                axios.get('/admin/branches',{params: params}).then(function(response) {
                    self.options.branches  = response.data.data.data;
                    self.branches_totalItems = response.data.data.total;
                    self.pagination.branches_totalItems = response.data.data.total;
                    (cb || Function)();
                });
            },
            loadCardLogin(cb) {

                const self = this;

                axios.get('/admin/cardlogin/'+self.id).then((res) => {
                    let cardLog = res.data.data;
                    self.key = cardLog.key;
                    self.branch = cardLog.branch_id;
                    self.id = cardLog.id;
                    console.log(cardLog);
                    
                    let safeDatetime = this.convertDateToString(cardLog.updated_at);

                    let datetime = safeDatetime.split(" ");

                    self.date = datetime[0];
                    self.time = datetime[1];
                });
            },
        }
    }
</script>