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

    <label for="sort">Sort By:</label>
    <select name="sort" v-model="selectedSort">
      <option value="date">Date</option>
      <option value="name">Name</option>
    </select>

    <label for="sort">Role</label>
    <select name="sort" v-model="selectedRole">
      <option value="All">All</option>
      <option value="Auditor">Auditor</option>
      <option value="User">User</option>
    </select>

    <table class="table" v-if="usersInPage">
      <thead>
        <tr>
          <th scope="col">#</th>
          <th scope="col">Name</th>
          <th scope="col">Role</th>
          <th scope="col">Actions</th>
        </tr>
      </thead>
      <tbody>
        <tr v-for="(user, index) in usersInPage" :key="user.id">
          <th scope="row">{{ currentPage * 10 + index + 1 }}</th>
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

    <div class="btn-toolbar" role="toolbar">
      <div class="btn-group me-2" role="group">
        <button
          type="button"
          class="btn btn-outline-primary"
          @click="currentPage--"
          :disabled="currentPage == 0"
        >
          Prev
        </button>
        <button type="button" class="btn btn-outline-primary" disabled>
          {{ currentPage + 1 + " of " + totalPages }}
        </button>
        <button
          type="button"
          class="btn btn-outline-primary"
          @click="currentPage++"
          :disabled="currentPage == totalPages - 1"
        >
          Next
        </button>
      </div>
    </div>

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
      selectedSort: "date",
      selectedRole: "All",
      currentPage: 0,
    };
  },
  watch: {
    totalPages(oldVal, newVal) {
      if (oldVal != newVal) {
        this.currentPage = 0;
      }
    },
  },
  computed: {
    ...mapState({
      account: (state) => state.account,
      users: (state) => state.users.all,
    }),
    usersToDisplay() {
      const originalUsersList = this.users.items || [];

      if (!Array.isArray(originalUsersList)) {
        return [];
      }

      let allUsers = [...originalUsersList];

      if (this.selectedSort == "name") {
        allUsers.sort((user1, user2) => {
          const user1Name = (user1.firstName + user1.lastName).toLowerCase();
          const user2Name = (user2.firstName + user2.lastName).toLowerCase();

          if (user1Name < user2Name) return -1;
          if (user1Name > user2Name) return 1;
          return 0;
        });
      }

      if (this.selectedRole != "All") {
        allUsers = allUsers.filter(({ role }) => role == this.selectedRole);
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
    totalPages() {
      return Math.ceil(this.usersToDisplay.length / 10);
    },
    usersInPage() {
      const currentIndex = this.currentPage * 10;
      return this.usersToDisplay.slice(currentIndex, currentIndex + 10);
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