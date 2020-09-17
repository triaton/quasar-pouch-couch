<template>
  <q-page>
    <div class="q-pa-md">
      <div class="row">
        <div class="col-10">
          <q-input label="todolist item" v-model="todoInput"/>
        </div>
        <div class="col-2 flex vertical-middle">
          <q-btn @click="addTodoList" :disabled="!todoInput">Add</q-btn>
        </div>
      </div>
      <div>
        <div v-for="(todoItem, index) in todoList" v-bind:key="index">
          <div>{{todoItem.text}}</div>
        </div>
      </div>
    </div>
  </q-page>
</template>

<script>
import PouchDB from 'pouchdb'
PouchDB.plugin(require('pouchdb-find'))
export default {
  data () {
    return {
      todoList: [],
      localDb: null,
      remoteDb: null,
      todoInput: ''
    }
  },
  methods: {
    addTodoList () {
      const todoItem = { _id: new Date().toISOString(), text: this.todoInput }
      this.todoList = [...this.todoList, todoItem]
      this.todoInput = ''
      this.localDb.put(todoItem)
      this.localDb.sync(this.remoteDb)
    },
    updateList () {
      this.localDb.allDocs({ include_docs: true }, (err, result) => {
        if (err) {
          console.log(err)
        }
        this.todoList = result.rows.map(row => row.doc)
      })
    }
  },
  mounted () {
    this.localDb = new PouchDB('localdb')
    this.remoteDb = new PouchDB('http://server.couchdb-api.com/db')
    this.localDb.replicate.from(this.remoteDb)
    this.updateList()
  }
}
</script>
