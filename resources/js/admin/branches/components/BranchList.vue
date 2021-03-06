<template>
    <div class="component-wrap">

        <!-- search -->
        <v-card class="pt-3">
            <div class="d-flex flex-row">
                <div class="flex-grow-1 pa-2">
                    <v-btn @click="$router.push({name:'branches.create'})" class="specialPrimary lighten-1" dark>
                        {{ translate('branch.new_branch')}}
                        <v-icon right dark>add</v-icon>
                    </v-btn>
                </div>
            </div>
            <div class="d-flex flex-lg-row flex-sm-column">
                <div class="flex-grow-1 pa-2">
                    <v-text-field filled prepend-icon="search" v-bind:label="translate('common.filter_by_tag')"  v-model="filters.tag"></v-text-field>
                </div>
            </div>
        </v-card>
        <!-- /search -->
        <v-divider class="pb-2"/>

        <!-- data table -->
        <v-data-table
                hide-default-header
                v-bind:headers="headers"
                :options.sync="pagination"
                :items="items"
                :server-items-length="totalItems"
                class="elevation-1">
            <template v-slot:header="{props:{headers}}">
                <thead>
                <tr>
                    <th v-for="header in headers">
                        <div v-if="header.value=='tag'" :class="`text-${header.align}`"><v-icon>mdi-person</v-icon> {{header.text}}</div>
                       <div v-else :class="`text-${header.align}`">{{header.text}}</div>
                    </th>
                </tr>
                </thead>
            </template>
            <template v-slot:body="{items}">
                <tbody>
                    <tr v-for="item in items" :key="item.id">
                        <td>
                            <div class="ml-n1 my-1 d-flex justify-space-between align-content-space-around flex-wrap">
                                <v-btn @click="$router.push({name:'branches.edit',params:{id: item.id}})" class="ma-1" small outlined icon color="info">
                                    <v-icon small>mdi-pencil</v-icon>
                                </v-btn>
                                <v-btn @click="trash(item)" class="ma-1" small outlined icon color="red">
                                    <v-icon small>mdi-delete</v-icon>
                                </v-btn>
                            </div>
                        </td>
                        <td>{{ item.id }}</td>
                        <td>{{ item.tag }}</td>
                        <td>{{ convertDateToString( item.created_at) }}</td>
                        <td>{{ convertDateToString( item.updated_at) }}</td>
                    </tr>
                </tbody>
            </template>
        </v-data-table>
    </div>
</template>

<script>
    export default {
        data () {
            return {
                headers: [
                    { text: this.translate('common.action'), value: false, align: 'left', sortable: false },
                    { text:  this.translate('common.id'), value: 'id', align: 'left', sortable: false },
                    { text: this.translate('common.tag'), value: 'name', align: 'left', sortable: false },
                    { text:  this.translate('common.created_at'), value: 'created_at', align: 'left', sortable: false },
                    { text:  this.translate('common.updated_at'), value: 'updated_at', align: 'left', sortable: false },
                ],
                items: [],
                totalItems: 0,
                pagination: {
                    itemsPerPage: 10
                },

                filters: {
                    name: '',
                },

                dialogs: {
                    showPermissions: {
                        items: [],
                        show: false
                    }
                }
            }
        },
        mounted() {
            const self = this;

            self.loadBranches(()=>{});

            self.$eventBus.$on(['BRANCH_ADDED','BRANCH_UPDATED','BRANCH_DELETED'],()=>{
                self.loadBranches(()=>{});
            });

            self.$store.commit('setBreadcrumbs',[
                            {label:this.translate('common.branches_list'),name:''}
            ]);
        
        },
        watch: {
            pagination: {
                handler () {
                    this.loadBranches(()=>{});
                },
            },
            'filters.key':_.debounce(function(){
                const self = this;
                self.loadBranches(()=>{});
            },700),
        },
        methods: {
   
        loadBranches(cb) {

                const self = this;

                let params = {
                    tag: self.filters.tag,
                    page: self.pagination.page,
                    per_page: self.pagination.itemsPerPage
                };

                axios.get('/admin/branches',{params: params}).then(function(response) {
                    self.items = response.data.data.data;
                    self.totalItems = response.data.data.total;
                    self.pagination.totalItems = response.data.data.total;
                    (cb || Function)();
                });
            },
              trash(item) {
                const self = this;

                self.$store.commit('showDialog',{
                    type: "confirm",
                    icon: 'warning',
                    title: this.translate('common.confirm_deletion'),
                    message: this.translate('common.are_you_sure_for_delete_this_user'),
                    okCb: ()=>{

                        axios.delete('/admin/branches/' + item.id).then(function(response) {

                            self.$store.commit('showSnackbar',{
                                message: response.data.message,
                                color: 'success',
                                duration: 3000
                            });

                            self.$eventBus.$emit('BRANCH_DELETED');

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
                    cancelCb: ()=>{
                        console.log("CANCEL");
                    }
                });
            },
        },
      
    }
</script>
