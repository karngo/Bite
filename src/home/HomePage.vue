<template>
  <div>
    <h1>Hi {{ account.user.firstName }}!</h1>
    <p>You're logged in!!</p>
    <h3>Users from secure api end point:</h3>
    <em v-if="users.loading">Loading users...</em>
    <span v-if="users.error" class="text-danger">ERROR: {{ users.error }}</span>

    <input
      type="text"
      v-model="searchString"
      name="search"
      placeholder="Search"
    />
    <table class="table" v-if="usersToDisplay">
      <thead>
        <tr>
          <th scope="col">#</th>
          <th scope="col">Name</th>
          <th scope="col">Role</th>
          <th scope="col">Actions</th>
        </tr>
      </thead>
      <tbody>
        <tr v-for="(user, index) in usersToDisplay" :key="user.id">
          <th scope="row">{{ index + 1 }}</th>
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
  data() {
    return {
      searchString: "",
    };
  },
  computed: {
    ...mapState({
      account: (state) => state.account,
      users: (state) => state.users.all,
    }),
    usersToDisplay() {
      const allUsers = this.users.items || [];

      if (!Array.isArray(allUsers)) {
        return [];
      }

      if (!this.searchString) {
        return allUsers;
      }

      const searchedUsers = allUsers.filter((user) => {
        const fullName = user.firstName + user.lastName;
        return fullName.toLowerCase().includes(this.searchString.toLowerCase());
      });

      return searchedUsers;
    },
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