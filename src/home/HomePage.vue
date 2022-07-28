<template>
  <div>
    <ul class="nav nav-pills mb-4">
      <li class="nav-item">
        <a
          class="nav-link"
          :class="{ active: activeTab == 'dashboard' }"
          @click="activeTab = 'dashboard'"
        >
          Dashboard
        </a>
      </li>
      <li class="nav-item">
        <a
          class="nav-link"
          :class="{ active: activeTab == 'audit' }"
          @click="activeTab = 'audit'"
          >Audit</a
        >
      </li>
      <li class="nav-item">
        <router-link
          class="nav-link"
          :class="{ active: activeTab == 'logout' }"
          @click="activeTab = 'logout'"
          to="/login"
        >
          Logout
        </router-link>
      </li>
    </ul>

    <div v-show="activeTab == 'dashboard'">
      <h1>Hi {{ account.user.firstName }}!</h1>
      <p>You're logged in!!</p>
      <h3>Users from secure api end point:</h3>
      <em v-if="users.loading">Loading users...</em>
      <span v-if="users.error" class="text-danger"
        >ERROR: {{ users.error }}</span
      >
    </div>

    <div v-show="activeTab == 'audit'">
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

      <label for="sort">Time</label>
      <select name="sort" v-model="timeFormat">
        <option value="24hr">24hr</option>
        <option value="12hr">12hr</option>
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
            <th scope="col">Created On</th>
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
              {{ user.createdDate | formatDate(timeFormat) }}
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
    </div>
  </div>
</template>

<script>
import { mapState, mapActions } from "vuex";

const getTwoDigit = (num) => ("0" + num).slice(-2);

export default {
  data() {
    return {
      searchString: "",
      selectedSort: "date",
      selectedRole: "All",
      currentPage: 0,
      activeTab: "dashboard",
      timeFormat: "24hr",
    };
  },
  filters: {
    formatDate(dateString, format) {
      const dateObj = new Date(dateString);
      const DD = getTwoDigit(dateObj.getDate());
      const MM = getTwoDigit(dateObj.getMonth() + 1);
      const YYYY = dateObj.getFullYear();
      const hh = getTwoDigit(dateObj.getHours() + 1);
      const mm = getTwoDigit(dateObj.getMinutes() + 1);
      const ss = getTwoDigit(dateObj.getSeconds() + 1);

      if (format == "12hr") {
        const hh12 = getTwoDigit(hh > 12 ? hh - 12 : hh);
        const ampm = hh > 11 ? "pm" : "am";
        return `${DD}/${MM}/${YYYY} ${hh12};${mm}:${ss}${ampm}`;
      }

      return `${DD}/${MM}/${YYYY} ${hh};${mm}:${ss}`;
    },
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
  mounted() {
    this.setActiveTab();
  },
  methods: {
    ...mapActions("users", {
      getAllUsers: "getAll",
      deleteUser: "delete",
    }),
    setActiveTab() {
      const loggedInUser = this.account.user;
      if (loggedInUser.role == "Auditor") {
        this.activeTab = "audit";
      }
    },
  },
};
</script>