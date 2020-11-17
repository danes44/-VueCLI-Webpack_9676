<template>
    <v-main class="list">
        <h3 class="text-h3 font-weight-medium mb-5">To Do List</h3>

        <v-card>
            <v-card-title>
                <v-text-field
                    v-model="search"
                    append-icon="mdi-magnify"
                    label="Search"
                    single-line
                    hide-details>
                </v-text-field>
                <v-spacer></v-spacer>
                <v-select
                    v-model="sortTodo"
                    :items="['Penting', 'Biasa', 'Tidak penting']"
                    label="Sort Priority"
                    dense
                    outlined
                    hide-details
                    single-line
                    class="pr-5"
                >
                </v-select>
                <v-btn color="success" dark @click="dialog = true">Tambah</v-btn>
            </v-card-title>

            <template>
                <v-data-table 
                    v-model="selected"
                    :headers="headers" 
                    :items="todos" 
                    :search="search" 
                    :expanded.sync="expanded"
                    
                    item-key="task"
                    show-expand
                    show-select
                    class="elevation-1"
                >
                    <template v-slot:[`item.priority`]="{ item }">
                        <v-chip :color="getColor(item)" outlined>
                            {{ item.priority }}
                        </v-chip>
                    </template>

                    <template v-slot:[`item.actions`]="{ item }">
                        
                        <v-btn small text icon class="mr-2" @click="editItem(item)"><v-icon color="blue">{{ icons.mdiPencil }}</v-icon></v-btn>
                        <v-btn small text icon @click="deleteItem(item)"><v-icon  color="red">{{ icons.mdiDelete }}</v-icon></v-btn>
                    </template>
                    <template v-slot:expanded-item="{ headers, item }">
                        <td :colspan="headers.length" class="text-left pa-4">
                            <h3>Note</h3>
                            {{ item.note }}
                        </td>
                    </template>
                </v-data-table>
            </template>
        </v-card>

        <v-card v-if="selected.length" class="text-left mt-5">
            <v-card-text >
                <v-container>
                    <b>Multiple Delete:</b>
                    <ul v-for="(select,i) in selected" :key="i">
                        <li>{{ select.task }}</li>
                    </ul>
                </v-container>
            </v-card-text>
            <v-card-actions>
                <v-btn color="red" class="ml-5 mb-5" outlined dense @click="deleteMultiple">Delete</v-btn>
            </v-card-actions>
        </v-card>

        <v-dialog v-model="dialog" persistent max-width="600px">
            <v-card>
                <v-card-title>
                    <span class="headline">Form Todo</span>
                </v-card-title>

                <v-card-text>
                    <v-container>
                        <v-text-field
                            v-model="formTodo.task"
                            label="Task"
                            required>
                        </v-text-field>
                        <v-select
                            v-model="formTodo.priority"
                            :items="['Penting', 'Biasa', 'Tidak penting']"
                            label="Priority"
                            required>
                        </v-select>
                        <v-textarea
                            v-model="formTodo.note"
                            label="Note"
                            required>
                        </v-textarea>
                    </v-container>
                </v-card-text>

                <v-card-actions>
                    <v-spacer></v-spacer>
                    <v-btn color="blue darken-1" text @click="cancel">Cancel</v-btn>
                    <v-btn color="blue darken-1" text @click="save">Save</v-btn>
                </v-card-actions>
            </v-card>
        </v-dialog>
        
        <v-dialog v-model="dialogEdit" persistent max-width="600px">
            <v-card>
                <v-card-title>
                    <span class="headline">Form Todo Edit</span>
                </v-card-title>
                
                <v-card-text>
                    <v-container>
                        <v-text-field
                            v-model="editedItem.task"
                            label="Task"
                            required>
                        </v-text-field>

                        <v-select
                            v-model="editedItem.priority"
                            :items="['Penting', 'Biasa', 'Tidak penting']"
                            label="Priority"
                            required>
                        </v-select>

                        <v-textarea
                            v-model="editedItem.note"
                            label="Note"
                            required>
                        </v-textarea>
                    </v-container>
                </v-card-text>

                <v-card-actions>
                    <v-spacer></v-spacer>
                    <v-btn color="blue darken-1" text @click="cancel">Cancel</v-btn>
                    <v-btn color="blue darken-1" text @click="saveEdit">Edit</v-btn>
                </v-card-actions>
            </v-card>
        </v-dialog>
        
        <v-dialog v-model="dialogDelete" persistent max-width="600px">
            <v-card>
                <v-card-title class="headline text-center">Are you sure you want to delete this item?</v-card-title>
                <v-card-actions>
                    <v-spacer></v-spacer>
                    <v-btn color="blue darken-1" text @click="closeDelete">Cancel</v-btn>
                    <v-btn color="red darken-1" text @click="deleteItemConfirm">OK</v-btn>
                    <v-spacer></v-spacer>
                </v-card-actions>
            </v-card>
        </v-dialog>

        
    </v-main>
</template>

<script>
    import {
        mdiAccount,
        mdiPencil,
        mdiShareVariant,
        mdiDelete,
    } from '@mdi/js'
    export default {
        name: "List",
        data() {
            return {
                icons: {
                    mdiAccount,
                    mdiPencil,
                    mdiShareVariant,
                    mdiDelete,
                },
                search: null,
                dialog: false,
                dialogEdit: false,
                dialogDelete: false,
                expanded: [],
                singleExpand: false,
                headers: [
                    {
                        text: "Task",
                        align: "start",
                        sortable: true,
                        value: "task",
                    },
                    { text: "Priority", value: "priority" },
                    { text: "Actions", value: "actions" },
                ],
                todos: [
                    {
                        task: "bernafas",
                        priority: "Penting",
                        note: "huffttt",
                    },
                    {
                        task: "nongkrong",
                        priority: "Tidak penting",
                        note: "bersama tman2",
                    },
                    {
                        task: "masak",
                        priority: "Biasa",
                        note: "masak air 500ml",
                    },
                ],
                formTodo: {
                    task: null,
                    priority: null,
                    note: null,
                },
                editedIndex: -1,
                editedItem: {
                    task: null,
                    priority: null,
                    note: null,
                },
                selected: [],
                tempIndex: null,
            };
        },
        methods: {
            save() {
                this.todos.push(this.formTodo);
                this.resetForm();
                this.dialog = false;
            },
            saveEdit() {
                // this.item.task = formTodo.task;
                // this.item.priority = formTodo.priority;
                // this.item.note = formTodo.note;
                Object.assign(this.todos[this.editedIndex], this.editedItem)
                this.dialogEdit = false;
            },
            cancel() {
                this.resetForm();
                this.dialog = false;
                this.dialogEdit = false;
            },
            resetForm() {
                this.formTodo = {
                    task: null,
                    priority: null,
                    note: null,
                };
            },
            deleteItem(item){
                this.dialogDelete = true;
                this.item = item;
            },
            deleteItemConfirm(){
                this.todos.splice(this.todos.indexOf(this.item), 1);
                this.dialogDelete = false;
            },
            closeDelete(){
                this.dialogDelete = false;
            },
            editItem(item){
                this.formTodo = {
                    task: item.task,
                    priority: item.priority,
                    note: item.note,
                }
                this.item = item;
                this.editedIndex = this.todos.indexOf(item)
                this.editedItem = Object.assign({}, item)
                this.dialogEdit = true;
            },
            getColor(item){
                if(item.priority == 'Penting')
                {
                    return "red";
                }
                else if(item.priority == 'Biasa')
                {
                    return "green";
                }
                else
                {
                    return "blue";
                }
            },
            deleteMultiple(){
                for(var i=0; i<this.selected.length; i++){
                    this.tempIndex = this.todos.indexOf(this.selected[i]); //nyari todos dengan index yang sama kayak selected pada index i trus masukin ke tempIndex
                    this.todos.splice(this.tempIndex,1); //delete pada index tersebut
                }
                this.selected = []
            }
        },
    };
</script>
