<template>
  <div>
    <h1>Hi {{ account.user.firstName }}!</h1>
    <p>You're logged in!!</p>
    <h3>Users from secure api end point:</h3>
    <em v-if="users.loading">Loading users...</em>
    <span v-if="users.error" class="text-danger">ERROR: {{ users.error }}</span>
    <table class="table" v-if="users.items">
      <thead>
        <tr>
          <th scope="col">Name</th>
          <th scope="col">Role</th>
          <th scope="col">Actions</th>
        </tr>
      </thead>
      <tbody>
        <tr v-for="user in users.items" :key="user.id">
          <td>
            {{ user.firstName + " " + user.lastName }}
          </td>
          <td>
            <span class="badge badge-secondary">{{ user.role }}</span>
          </td>
          <td>
            <span v-if="user.role == 'Auditor'">
              <a @click="deleteUser(user.id)" class="badge badge-danger"
                >Delete</a
              ></span
            >
          </td>
        </tr>
      </tbody>
    </table>
    <p>
      <router-link to="/login">Logout</router-link>
    </p>
  </div>
</template>

<script>
import { mapState, mapActions } from "vuex";

export default {
  computed: {
    ...mapState({
      account: (state) => state.account,
      users: (state) => state.users.all,
    }),
  },
  created() {
    this.getAllUsers();
  },
  methods: {
    ...mapActions("users", {
      getAllUsers: "getAll",
      deleteUser: "delete",
    }),
  },
};
</script>