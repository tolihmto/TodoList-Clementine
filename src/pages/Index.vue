<template>
  <Layout>
    <b-navbar class="is-link" :fixed-top="true" :spaced="true">
        <template slot="brand">
          <strong style="font-size: 30px">
            <g-link class="my-brand" to="/">{{ $static.metadata.siteName }}</g-link>
          </strong>
        </template>
        <template slot="end">
          <b-field class="buttons">
            <b-button
                label="Désélectionner"
                type="is-danger"
                icon-left="close"
                :disabled="!selected"
                @click="selected = null" />
            <b-button
                label="Ajouter une todo"
                type="is-primary"
                @click="addTask()" />
            <b-dropdown :triggers="['hover']" aria-role="list">
            <template #trigger>
                <b-button
                    style="margin-right: 8px"
                    label="Modifier la sélection"
                    type="is-primary"
                    icon-right="menu-down" />
                </template>
                <b-dropdown-item aria-role="listitem" @click="changeTask()">Changer le titre</b-dropdown-item>
                <b-dropdown-item aria-role="listitem" @click="changeStatus()">Valider la todo</b-dropdown-item>
            </b-dropdown>
            <b-button type="is-primary" @click="removeTask()">Suprimer une todo</b-button>
            <b-button type="is-primary" @click="removeTasks()">Suprimer les todos</b-button>
          </b-field>
        </template>
    </b-navbar>
    
    <!-- CODE MIT EN COMMENTAIRE CAR C'EST UNE ANCIENNE VERSION UTILISANT L'API DATA STORE -->
    
    <!-- <div class="todoList">
      <div class="todo" v-for="edge in $page.allTasks.edges" :key="edge.node.id">
        <div class="todo-content columns">
          <div class="column">
            <p class="line">{{ edge.node.userId }}</p>
            <p class="line">{{ edge.node.id }}</p>
            <p class="line num">Todo n°{{ edge.node.id }}</p>
            <p class="line">{{ edge.node.completed }}</p>
          </div>
        </div>
        <div class="todo-content columns">
          <div class="column">
            <p class="line">{{ edge.node.userId }}</p>
            <p class="line">{{ edge.node.id }}</p>
            <p class="line" v-bind:class="{ green: edge.node.completed, red: !edge.node.completed }">{{ edge.node.title }}</p>
            <p class="line">{{ edge.node.completed }}</p>
          </div>
        </div>
      </div>
    </div> -->
    <section>
        <b-tabs>
            <b-tab-item label="Table">
                <b-table
                    :data="list"
                    :columns="columns"
                    :selected.sync="selected"
                    focusable>
                </b-table>
            </b-tab-item>

            <b-tab-item label="Selected">
                <pre>{{ selected }}</pre>
            </b-tab-item>
        </b-tabs>
    </section>
  </Layout>
</template>

<static-query>
query {
  metadata {
    siteName
  }
}
</static-query>

<page-query>
query allTasks {
  allTasks(sortBy: "id", order: ASC) {
    edges {
      node {
        userId,
        id,
        title,
        completed
      }
    }
  }
}
</page-query>

<script>
import axios from 'axios'

export default {
  data() {
    return {
      list: [],
      selected: {},
        columns: [
          {
              field: 'id',
              label: 'ID',
              width: '40',
              numeric: true
          },
          {
              field: 'title',
              label: 'Todos',
              centered: true
          },
          {
              field: 'completed',
              label: 'Completed',
              width: '50'
          }
        ]
    }
  },
  async mounted() {
    this.apiRead()
  },
  metaInfo: {
    title: 'Home'
  },
  methods: {
    apiCreate(val) {
      let post = {
        title: val,
        body: 'body',
        userId: 1
      };
      axios.post("https://jsonplaceholder.typicode.com/todos", post).then((result) => {
        console.log(result);
      });
    },
    async apiRead() {
      try {
        const results = await axios.get(
          'https://jsonplaceholder.typicode.com/todos'
        )

        this.list = results.data
        this.selected = this.list[0]
      } catch (error) {
        console.log(error)
      }
    },
    apiUpdate(id) {
      let put = {
        id: id,
        title: 'new title',
        body: 'new body',
        userId: 1
      };
      axios.put(`https://jsonplaceholder.typicode.com/todos/${id}`, put).then((result) => {
        console.log(result);
      });
    },
    apiDelete(id) {
      axios.delete(`https://jsonplaceholder.typicode.com/todos/${id}`).then((result) => {
        console.log(result);
      });
    },
    changeTask() {
      this.$buefy.dialog.prompt({
          message: `Nouveau titre ?`,
          inputAttrs: {
              placeholder: this.selected.title,
              maxlength: 100
          },
          trapFocus: true,
          onConfirm: (value) => this.apiUpdate(this.list.length) & (this.list.splice(this.selected.id - 1, 1, {id: this.selected.id, title: `${value}`, userId: 1, completed: false}))
      })
    },
    changeStatus() {
      this.$buefy.dialog.confirm({
          title: 'Voulez-vous valider cette todo ?',
          message: ``,
          cancelText: 'Annuler',
          confirmText: 'Valider',
          type: 'is-success',
          onConfirm: () => this.apiUpdate(this.list.length) & (this.list.splice(this.selected.id - 1, 1, {id: this.selected.id, title: this.selected.title, userId: this.selected.userId, completed: true})) & this.$buefy.toast.open('La Todo est validée')
      })
    },
    addTask() {
      this.$buefy.dialog.prompt({
          message: `Qu'avez à faire ?`,
          inputAttrs: {
              placeholder: 'something crazy',
              maxlength: 100
          },
          trapFocus: true,
          onConfirm: (value) => this.apiCreate(value) & (this.list.push({id: this.list.length + 1, title: `${value}`, userId: 1, completed: false}))
      })
    },
    removeTask() {
      this.list.pop() & this.apiDelete(this.list.length + 1)
    },
    removeTasks() {
      this.list = []
    }
  }
}
</script>

<style>

nav.navbar.is-fixed-top {
  transition: background-color 0.5s ease;
  background: rgb(51, 51, 51);
}

.my-brand:hover {
  color: white;
}

.home-links a {
  margin-right: 1rem;
}
</style>
