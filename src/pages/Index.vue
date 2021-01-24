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
                label="Clear selected"
                type="is-danger"
                icon-left="close"
                :disabled="!selected"
                @click="selected = null" />
            <b-button type="is-primary">Ajouter une todo</b-button>
          </b-field>
        </template>
    </b-navbar>
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
        ]
    }
  },
  async mounted() {
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
  metaInfo: {
    title: 'Home'
  },
  methods: {
    clickMe() {
      this.$buefy.notification.open('Clicked!!')
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
