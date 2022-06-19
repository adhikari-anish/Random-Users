<template>
  <div class="table-container">
    <h1 class="table-title">Random Users Table</h1>
    <table class="table">
      <thead class="table__head">
        <tr>
          <th
            class="table__head--sn"
            @click="sortBy('sn')"
            :class="sortClass('sn')"
          >
            S.N.
          </th>
          <th
            class="table__head--name"
            @click="sortBy('fullName')"
            :class="sortClass('fullName')"
          >
            Name
          </th>
          <th
            class="table__head--country"
            @click="sortBy('country')"
            :class="sortClass('country')"
          >
            Country
          </th>
          <th
            class="table__head--dob"
            @click="sortBy('dob')"
            :class="sortClass('dob')"
          >
            DOB
          </th>
        </tr>
      </thead>
      <tbody class="table__body">
        <tr v-if="loading">
          <td colspan="4" class="table-loader">Loading...</td>
        </tr>
        <tr v-else v-for="data in sortedUsers" :key="data.id.value">
          <td>{{ data.sn }}</td>
          <td class="table__data--name">
            <img
              :src="data.picture.large"
              :alt="data.fullName"
              width="50"
              height="50"
            />
            <span>
              {{ `${data.nameTitle} ${data.fullName}` }}
            </span>
          </td>
          <td>{{ data.country }}</td>
          <td>{{ data.dob }}</td>
        </tr>
        <tr v-if="sortedUsers?.length === 0">
          <td colspan="4">No any users</td>
        </tr>
      </tbody>
    </table>
  </div>
</template>

<script>
export default {
  name: "HomeView",

  data: function () {
    return {
      users: [],
      loading: false,
      sort: {
        key: "",
        isAsc: false,
      },
    };
  },

  mounted() {
    this.fetchUsers();
  },

  computed: {
    transformedUsers() {
      if (this.users.length) {
        return this.users.map((user, index) => {
          return {
            ...user,
            sn: index + 1,
            nameTitle: user.name.title,
            fullName: `${user.name.first} ${user.name.last}`,
            dob: this.getDobYear(user.dob.date),
            country: user.location.country,
          };
        });
      }
    },
    sortedUsers() {
      const users = this.transformedUsers;
      if (!!this.sort.key) {

        users.sort((a, b) => {
          a = a[this.sort.key];
          b = b[this.sort.key];

          return (a === b ? 0 : a > b ? 1 : -1) * (this.sort.isAsc ? 1 : -1);
        });
      }
      return users;
    },
  },

  methods: {
    async fetchUsers() {
      this.loading = true;
      try {
        let usersData = await fetch("https://randomuser.me/api?results=10");
        let users = await usersData.json();
        this.users = users.results;
      } catch (err) {
        console.error(err);
      } finally {
        this.loading = false;
      }
    },
    getDobYear(date) {
      let d = new Date(date);
      return d.toLocaleDateString("en-CA");
    },
    sortBy(key) {
      this.sort.isAsc = this.sort.key === key ? !this.sort.isAsc : false;
      this.sort.key = key;
    },
    sortClass(key) {
      return this.sort.key === key
        ? `sorted ${this.sort.isAsc ? "asc" : "desc"}`
        : "";
    },
  },
};
</script>

<style scoped lang="scss">
.table {
  width: 100%;
  border-collapse: collapse;

  &__head {
    th {
      padding: 14px 8px;
      background: rgba(142, 155, 144, 0.5);
      text-align: left;
      cursor: pointer;
      user-select: none;

      &.sorted {
        &.asc::after {
          display: inline-block;
          content: "▼";
        }

        &.desc::after {
          display: inline-block;
          content: "▲";
        }
      }
    }

    &--sn {
      width: 20%;
    }

    &--name {
      width: 40%;
    }

    &--country {
      width: 20%;
    }

    &--dob {
      width: 20%;
    }
  }

  &__body {
    tr {
      &:nth-child(odd) {
        background-color: rgba(147, 192, 164, 0.5);

        &:hover {
          background-color: rgba(220, 226, 189, 0.5);
        }
      }

      &:nth-child(even) {
        background-color: rgba(182, 196, 162, 0.5);
        &:hover {
          background-color: rgba(220, 226, 189, 0.5);
        }
      }
    }

    td {
      padding: 14px 8px;
      text-align: left;
      vertical-align: middle;
    }
  }

  &__data {
    &--name {
      position: relative;
      img {
        border-radius: 50%;
      }

      span {
        margin-left: 8px;
        position: absolute;
        top: 50%;
        transform: translateY(-50%);
      }
    }
  }
}

.table {
  &-container {
    margin: 20px;
  }

  &-title {
    margin-bottom: 10px;
  }

  &-loader {
    text-align: center !important;
    font-size: 22px;
    font-weight: semi-bold;
  }
}
</style>
